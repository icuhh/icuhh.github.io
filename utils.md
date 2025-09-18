# 整理一些日常可能会用到且调试过的工具类



## base62编码

------



> 常用于短链接生成等场景

```
public class Base62Util {

    public static final String BASE62_CHARACTERS = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";

    private static final int BASE = 62;

    public static String encode(long number) {
        StringBuilder encoded = new StringBuilder();
        while (number > 0) {
            //计算余数
            int remainder = (int) (number % BASE);
            // 找到对应的字符并追加到结果中
            encoded.append(BASE62_CHARACTERS.charAt(remainder));
            //更新number
            number /= BASE;
        }
        // 反转字符串以获得正确的顺序
        return encoded.reverse().toString();
    }

    public static long decode(String encoded) {
        long number = 0;
        for (int i = 0; i < encoded.length(); i++) {
            char c = encoded.charAt(i);
            // 找到字符对应的索引
            int index = BASE62_CHARACTERS.indexOf(c);
            //更新number
            number = number * BASE + index;
        }
        return number;
    }
}
```



## 比较对象属性不同

------



> 常用于获取对象修改前后属性的不同值

```
public class CompareObjUtil {

    public static List<Comparison> compareObj(Object beforeObj, Object afterObj) throws Exception {
        List<Comparison> differents = new ArrayList<Comparison>();

        if (beforeObj == null) throw new Exception("原对象不能为空");
        if (afterObj == null) throw new Exception("新对象不能为空");
        if (!beforeObj.getClass().isAssignableFrom(afterObj.getClass())) {
            throw new Exception("两个对象不相同，无法比较");
        }

        //取出属性 - 这种方式拿不到父类的属性
//      Field[] beforeFields = beforeObj.getClass().getDeclaredFields();
//      Field[] afterFields = afterObj.getClass().getDeclaredFields();
        Field[] beforeFields = FieldUtils.getAllFields(beforeObj.getClass());
        Field[] afterFields = FieldUtils.getAllFields(beforeObj.getClass());
        Field.setAccessible(beforeFields, true);
        Field.setAccessible(afterFields, true);

        //遍历取出差异值
        if (beforeFields != null && beforeFields.length > 0) {
            for (int i = 0; i < beforeFields.length; i++) {
                Object beforeValue = beforeFields[i].get(beforeObj);
                Object afterValue = afterFields[i].get(afterObj);
                if ((beforeValue != null && !"".equals(beforeValue) && !beforeValue.equals(afterValue)) || ((beforeValue == null || "".equals(beforeValue)) && afterValue != null)) {
                    Comparison comparison = new Comparison();
                    comparison.setField(beforeFields[i].getName());
                    comparison.setBefore(beforeValue);
                    comparison.setAfter(afterValue);
                    differents.add(comparison);
                }
            }
        }

        return differents;
    }

    /**
     * 处理两个泛型修改后的值
     *
     * @param original 原始值
     * @param updated  修改后的值
     * @param category 分类名称
     * @param <T>      泛型
     * @return map
     */
    public static <T> Map<String, List<String>> compare(T original, T updated, String category) {
        Map<String, List<String>> changes = new HashMap<>();

        // 获取类的所有字段
        Field[] fields = FieldUtils.getAllFields(original.getClass());

        for (Field field : fields) {
            field.setAccessible(true); // 允许访问私有字段
            try {
                Object originalValue = field.get(original);
                Object updatedValue = field.get(updated);
                // 检查值是否不同，或者原始值为空且更新值不为空
                if (originalValue == null && updatedValue != null) {
                    // 原值为空，改后有值
                    ApiModelProperty annotation = field.getAnnotation(ApiModelProperty.class);
                    String fieldName = annotation != null ? annotation.value() : field.getName();
                    changes.computeIfAbsent(category, k -> new ArrayList<>())
                            .add(fieldName + " 原值: null 改后值: " + updatedValue);
                    continue;
                }

                if (originalValue != null && !originalValue.equals(updatedValue)) {
                    // 原值不为空且与改后值不同
                    ApiModelProperty annotation = field.getAnnotation(ApiModelProperty.class);
                    String fieldName = annotation != null ? annotation.value() : field.getName();
                    changes.computeIfAbsent(category, k -> new ArrayList<>())
                            .add(fieldName + " 原值: " + originalValue + " 改后值: " + updatedValue);
                }
            } catch (IllegalAccessException e) {
                e.printStackTrace();
                log.info("无法访问字段: ", field.getName());
            }
        }
        return changes;
    }

    /**
     * 合并元素
     *
     * @param sourceMap 被合并的值
     * @param targetMap 合并的值
     */
    public static void mergeMaps(Map<String, List<String>> sourceMap, Map<String, List<String>> targetMap) {
        for (Map.Entry<String, List<String>> entry : sourceMap.entrySet()) {
            String key = entry.getKey();
            List<String> changes = entry.getValue();

            // 检查目标 Map 是否已存在该键
            if (targetMap.containsKey(key)) {
                // 如果存在，则合并列表
                targetMap.get(key).addAll(changes);
            } else {
                // 如果不存在，则直接添加
                targetMap.put(key, new ArrayList<>(changes));
            }
        }
    }

    /**
     * 格式输出转成String
     *
     * @param map 修改后对比的值
     * @return String
     */
    public static String convertMapToString(Map<String, List<String>> map) {
        StringBuilder result = new StringBuilder();

        for (Map.Entry<String, List<String>> entry : map.entrySet()) {
            String category = entry.getKey();
            List<String> changes = entry.getValue();

            result.append(category).append(":\n");
            for (String change : changes) {
                result.append("  - ").append(change).append("\n");
            }
        }
        return result.toString();
    }

}
```

## 对象字段获取

------

> 常用于通过反射获取对象的属性值信息

```
public class FieldUtils {

    /**
     * 根据字段名称获取字段信息(包括父类)
     *
     * @param
     * @return
     */
    public static Field getField(Class<?> clazz, String fieldName) {
        try {
            Field field = clazz.getDeclaredField(fieldName); // 获取当前类声明的字段
            field.setAccessible(true); // 设置字段可访问，即使是私有字段也可以获取到
            return field;
        } catch (NoSuchFieldException e) {
            // 当前类不存在该字段，尝试从父类中获取
            Class<?> superClass = clazz.getSuperclass();
            if (superClass == null) {
//                throw new IllegalArgumentException("Field not found: " + fieldName);
                return null;
            } else {
                return getField(superClass, fieldName);
            }
        }
    }

    /**
     * 获取指定类中所有的字段(包括父类)
     *
     * @param cls 类名
     * @return 字段组成的数组
     */
    public static Field[] getAllFields(Class<?> cls) {
        List<Field> allFieldsList = getAllFieldsList(cls);
        return (Field[]) allFieldsList.toArray(new Field[allFieldsList.size()]);
    }

    /**
     * 获取指定类中的所有字段
     *
     * @param cls 类名
     * @return 字段组成的集合
     */
    public static List<Field> getAllFieldsList(Class<?> cls) {
        Validate.isTrue(cls != null, "The class must not be null", new Object[0]);
        List<Field> allFields = new ArrayList();

        for (Class currentClass = cls; currentClass != null; currentClass = currentClass.getSuperclass()) {
            Field[] declaredFields = currentClass.getDeclaredFields();
            Collections.addAll(allFields, declaredFields);
        }
        return allFields;
    }

    /**
     * 获取指定类中的所有字段
     *
     * @param cls 类名
     * @return 字段组成的Map(key = 字段名 ， value = 字段)
     */
    public static Map<String, Field> getALlFieldMap(Class<?> cls) {
        Map<String, Field> fieldMap = new HashMap<>();
        List<Field> allFieldsList = getAllFieldsList(cls);
        if (CollectionUtils.isNotEmpty(allFieldsList)) {
            fieldMap = allFieldsList.stream().collect(Collectors.toMap(Field::getName, field -> {
                field.setAccessible(true);
                return field;
            }));
        }
        return fieldMap;
    }


    /**
     * 根据类名获取对应TableName注解中的值
     *
     * @param
     * @return
     */
    public static String getTableName(Class<?> cls) {
        String tableName = null;
        for (Class currentClass = cls; currentClass != null; currentClass = currentClass.getSuperclass()) {
            if (currentClass.isAnnotationPresent(TableName.class)) {
                TableName annotation = (TableName) currentClass.getAnnotation(TableName.class);
                tableName = annotation.value();
                break;
            }
        }
        return tableName;
    }

    /**
     * 获取具体字段数据源
     *
     * @param field 类字段
     * @param data  数据源
     * @return Object
     */
    public static Object getFieldValue(Field field, Object data) {
        try {
            Object value = field.get(data);
            if (value != null) {
                return value;
            }
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        return null;
    }


    /**
     * 验证集合某个字段的值是否一致
     *
     * @param list           源数据集合
     * @param fieldExtractor 字段提取函数
     */
    public static <T> boolean areFieldValuesSame(List<T> list, java.util.function.Function<T, ?> fieldExtractor) {
        if (list.isEmpty()) {
            return true;
        }
        Object firstValue = fieldExtractor.apply(list.get(0));
        for (int i = 1; i < list.size(); i++) {
            Object currentValue = fieldExtractor.apply(list.get(i));
            if (!Objects.equals(firstValue, currentValue)) {
                return false;
            }
        }
        return true;
    }

    /**
     * 复制属性
     *
     * @param source         源对象
     * @param target         目标对象
     * @param excludedFields 排除的字段
     */
    public static void copyProperties(Object source, Object target, String... excludedFields) {
        // 获取源对象和目标对象的类
        Class<?> sourceClass = source.getClass();
        Class<?> targetClass = target.getClass();

        // 遍历源对象的所有字段
        for (Field sourceField : getAllFieldsList(sourceClass)) {
            if (isFieldExcluded(sourceField.getName(), excludedFields)) {
                log.info("Excluding field: {}", sourceField.getName());
                continue;
            }
            try {
                sourceField.setAccessible(true);
                Object value = sourceField.get(source);
                Field targetField = getField(targetClass, sourceField.getName());
                if (null == targetField) {
                    continue;
                }
                targetField.setAccessible(true);
                targetField.set(target, value);
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 判断字段是否排除
     *
     * @param fieldName      字段名
     * @param excludedFields 排除字段数组
     * @return true 排除，false 不排除
     */
    private static boolean isFieldExcluded(String fieldName, String[] excludedFields) {
        for (String excluded : excludedFields) {
            if (fieldName.equals(excluded)) {
                return true;
            }
        }
        return false;
    }

}
```

## forEach 获取下标

------

> 常用用于循环获取下标

```
public class IndexGetUtils {
    public static <T> Consumer<T> getIndex(BiConsumer<T, Integer> consumer) {
        class IndexObject {
            int index;
        }
        IndexObject indexObject = new IndexObject();
        return i -> {
            consumer.accept(i, indexObject.index++);
        };
    }
}
```

## EXCEL图片读取

------

> 常用于excel 表格图片列上传获取并上传

```
public class ExcelReadImageUtil {

    @Autowired
    private OssUtils ossUtils;

    public <T> void readImage(InputStream inputStream, List<T> list) {
        try {
            Workbook workbook = WorkbookFactory.create(inputStream);
            // 默认读取第一页
            XSSFSheet sheet = (XSSFSheet) workbook.getSheetAt(0);
            List<POIXMLDocumentPart> documentPartList = sheet.getRelations();
            Integer size = list.size();
            for (POIXMLDocumentPart part : documentPartList) {
                if (part instanceof XSSFDrawing) {
                    XSSFDrawing drawing = (XSSFDrawing) part;
                    List<XSSFShape> shapes = drawing.getShapes();
                    for (XSSFShape shape : shapes) {
                        XSSFPicture picture = (XSSFPicture) shape;
                        XSSFClientAnchor anchor = picture.getPreferredSize();
                        CTMarker marker = anchor.getFrom();
                        int row = marker.getRow() - 1;
                        int col = marker.getCol();
                        //去掉表头， 从第1行开始
                        if (row > 0 && row <= size) {
                            PictureData pictureData = picture.getPictureData();
                            String extension = pictureData.suggestFileExtension();
                            byte[] bytes = pictureData.getData();
                            String uploadFile = ossUtils.uploadFile(bytes, "excelWrite", "." + extension);
                            if (StringUtils.isBlank(uploadFile)) {
                                continue;
                            }
                            String url = ossUtils.getUploadUrl(uploadFile);
                            //拿到当前行数据
                            T item = list.get(row - 1);
                            Class clazz = item.getClass();
                            Field[] fields = clazz.getDeclaredFields();
                            for (Field field : fields) {
                                if (field.isAnnotationPresent(ExcelImageProperty.class)) {
                                    ExcelImageProperty excelImageProperty = field.getAnnotation(ExcelImageProperty.class);
                                    int index = excelImageProperty.index();
                                    if (index == col) {
                                        field.setAccessible(true);
                                        Object obj = field.get(item);
                                        List<String> fileList = Objects.isNull(obj) ? new ArrayList<>() : Convert.toList(String.class, obj);
                                        fileList.add(url);
                                        field.set(item, fileList);
                                    }
                                }
                            }
                        }
                    }
                }
            }
        } catch (IOException | IllegalAccessException e) {
            e.printStackTrace();
            log.error("read image error {}", e);
        }
    }

}
```


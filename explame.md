# 处理过的一些需求技术点

> 关于一些在项目中开发实际用到的技术点例子



## EXCEL静态表头加动态插入表头字段导出

------

在导出的实体类中定义存储动态列表头的Map

```
    @ExcelProperty(value = "动态列")
    private Map<String, Object> dynamicData = new HashMap<>();
```

组装新增的动态表头

```
            //新增的动态表头
            List<ExportHeadVO> addHeadList = new ArrayList<>();
            //查询系统选项配置的维修实修字段，当作表头
            List<EmcInspectionProjectSaveDTO> optionProjectList = emcInspectionProjectMapper.listCheckPage(null, 1899023375988244483L);
            optionProjectList.forEach(x -> {
                if (StrUtil.isNotEmpty(x.getOptionContent())) {
                    EmcProjectContentDTO item = JSONUtil.toBean(x.getOptionContent(), EmcProjectContentDTO.class);
                    if (BeanUtil.isNotEmpty(item)) {
                        // 使用数组收集所有字段值
                        String[] values = new String[]{
                                item.getOne(), item.getTwo(), item.getThree(), item.getFour(), item.getFive(), item.getSix(),
                                item.getSeven(), item.getEight(), item.getNine(), item.getTen(), item.getEleven(),
                                item.getTwelve(), item.getThirteen(), item.getFourteen(), item.getFifteen(), item.getSixteen(),
                                item.getSeventeen(), item.getEighteen(), item.getNineteen(), item.getTwenty()
                        };
                        // 使用Stream处理数组
                        Arrays.stream(values).filter(StringUtils::isNotBlank).forEach(value -> {
                            ExportHeadVO headVO = new ExportHeadVO();
                            headVO.setKey("实际维修条目信息");
                            headVO.setName(x.getName() + "-" + value);
                            headList.add(headVO);
                            addHeadList.add(headVO);
                        });
                    } else {
                        ExportHeadVO headVO = new ExportHeadVO();
                        headVO.setKey("实际维修条目信息");
                        headVO.setName(x.getName());
                        headList.add(headVO);
                        addHeadList.add(headVO);
                    }
                }
            });
```

将动态表头跟数据匹配上

```
for (ExportHistoryRecordVO item : listPage) {
	            //导出实修内容
            if (CollectionUtil.isNotEmpty(addHeadList)) {
                // 为每条数据添加动态列值
                Map<String, Object> dynamicData = new HashMap<>();
                for (ExportHeadVO headVO : addHeadList) {
                    if (MapUtils.isNotEmpty(repairProjectValueMap) && repairProjectValueMap.containsKey(item.getId())) {
                        List<String> valueList = repairProjectValueMap.get(item.getId()).stream().flatMap(value -> {
                            List<String> projectValueList = new ArrayList<>();
                            if (StringUtils.isNotBlank(value.getOptionContent())) {
                                Arrays.stream(value.getOptionContent().split(",")).forEach(content -> {
                                    projectValueList.add(value.getName() + "-" + content);
                                });
                            } else {
                                projectValueList.add(value.getName());
                            }
                            return projectValueList.stream();
                        }).collect(Collectors.toList());
                        if (valueList.contains(headVO.getName())) {
                            dynamicData.put(headVO.getName(), "是");
                            continue;
                        }
                        //如果不满足，还有可能是输入框的
                        List<String> filterList = valueList.stream().filter(v -> v.split("-")[Constant.INTEGER_ZERO].equals(headVO.getName())).collect(Collectors.toList());
                        if (CollectionUtil.isNotEmpty(filterList)) {
                            dynamicData.put(headVO.getName(), filterList.get(Constant.INTEGER_ZERO).split("-")[Constant.INTEGER_ONE]);
                        } else {
                            dynamicData.put(headVO.getName(), "否");
                        }
                    }
                }
                item.setDynamicData(dynamicData);
            }
}
```

将组装好的表头设置进去

```
List<List<Object>> data = ExcelExportUtil.getInstance(dto.setHeadVOList(headList)).handleExport(listPage);
```




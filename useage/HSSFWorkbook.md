
# 使用

## 添加依赖

```xml
<!--    测试使用 XSSFWorkbook Excel操作工具    -->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>3.14</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>3.14</version>
</dependency>
```

## 添加Excel生成代码

```java
package com.mybatis.example;

import com.mybatis.example.entity.User;
import org.apache.poi.hssf.usermodel.*;
import org.apache.poi.hssf.util.HSSFColor;
import org.springframework.util.CollectionUtils;

import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.OutputStream;
import java.util.List;

public class WorkBookHelper {
    public void exportWorkBook(HttpServletResponse response, List<User> list) throws IOException {
        if (CollectionUtils.isEmpty(list)) {
            return;
        }

        String fileName = "AAA";

        HSSFWorkbook hssfWorkbook = new HSSFWorkbook(); //创建Excel工作薄对象
        HSSFSheet sheet = hssfWorkbook.createSheet(); //创建Excel工作表对象
        sheet.setColumnWidth(0, 8000);
        sheet.setColumnWidth(1, 5000);
        sheet.setColumnWidth(2, 4000);
        sheet.setColumnWidth(3, 2500);
        sheet.setColumnWidth(4, 3000);
        sheet.setColumnWidth(5, 6000);
        sheet.setColumnWidth(6, 6000);

        HSSFFont columnHeadFont = hssfWorkbook.createFont(); // 设置表头字体样式
        columnHeadFont.setFontName("宋体");
        columnHeadFont.setFontHeightInPoints((short) 10);
        columnHeadFont.setColor(HSSFColor.RED.index);
        columnHeadFont.setBoldweight(HSSFFont.BOLDWEIGHT_BOLD);

        // 列头的样式
        HSSFCellStyle columnHeadStyle = hssfWorkbook.createCellStyle();
        columnHeadStyle.setFont(columnHeadFont);
        // 左右居中
        columnHeadStyle.setAlignment(HSSFCellStyle.ALIGN_CENTER);
        // 上下居中
        columnHeadStyle.setVerticalAlignment(HSSFCellStyle.VERTICAL_CENTER);
        columnHeadStyle.setLocked(true);
        columnHeadStyle.setWrapText(true);
        // 左边框的颜色
        columnHeadStyle.setLeftBorderColor(HSSFColor.BLACK.index);
        // 边框的大小
        columnHeadStyle.setBorderLeft((short) 1);
        // 右边框的颜色
        columnHeadStyle.setRightBorderColor(HSSFColor.BLACK.index);
        // 边框的大小
        columnHeadStyle.setBorderRight((short) 1);
        // 设置单元格的边框为粗体
        columnHeadStyle.setBorderBottom(HSSFCellStyle.BORDER_THIN);
        // 设置单元格的边框颜色
        columnHeadStyle.setBottomBorderColor(HSSFColor.BLACK.index);
        // 设置单元格的背景颜色（单元格的样式会覆盖列或行的样式）
        columnHeadStyle.setFillForegroundColor(HSSFColor.WHITE.index);
        // 设置普通单元格字体样式
        HSSFFont font = hssfWorkbook.createFont();
        font.setFontName("宋体");
        font.setFontHeightInPoints((short) 10);

        //创建Excel工作表第一行
        HSSFRow row0 = sheet.createRow(0);
        row0.setHeight((short) 750);// 设置行高

        HSSFCell cell = row0.createCell(0);
        cell.setCellValue(new HSSFRichTextString("学生id"));//设置单元格内容
        cell.setCellStyle(columnHeadStyle);//设置单元格字体样式

        cell = row0.createCell(1);
        cell.setCellValue(new HSSFRichTextString("姓名"));
        cell.setCellStyle(columnHeadStyle);

        cell = row0.createCell(2);
        cell.setCellValue(new HSSFRichTextString("年龄"));
        cell.setCellStyle(columnHeadStyle);

        cell = row0.createCell(3);
        cell.setCellValue(new HSSFRichTextString("email"));
        cell.setCellStyle(columnHeadStyle);

        // 循环写入数据
        for (int i = 0; i < list.size(); i++) {
            User user = list.get(i);
            HSSFRow row = sheet.createRow(i+1);

            cell = row.createCell(0);
            cell.setCellValue(new HSSFRichTextString(String.valueOf(user.getId())));
            cell.setCellStyle(columnHeadStyle);

            cell = row.createCell(1);
            cell.setCellValue(new HSSFRichTextString(user.getName()));
            cell.setCellStyle(columnHeadStyle);

            cell = row.createCell(2);
            cell.setCellValue(new HSSFRichTextString(String.valueOf(user.getAge())));
            cell.setCellStyle(columnHeadStyle);

            cell = row.createCell(3);
            cell.setCellValue(new HSSFRichTextString(String.valueOf(user.getEmail())));
            cell.setCellStyle(columnHeadStyle);
        }

        OutputStream os = response.getOutputStream();// 获取输出流
        response.reset();// 重置输出流
        response.setHeader("Content-disposition",
                "attachment; filename=" + new String(fileName.getBytes("GB2312"), "8859_1") + ".xls");// 设定输出文件头
        response.setContentType("application/msexcel");// 定义输出类型

        hssfWorkbook.write(os);
        os.close();
        return;
    }
}
```

## 添加Restful请求

```java
@GetMapping("/getExcel")
public void getExcel(HttpServletRequest request, HttpServletResponse response){
    try {
        List<User> list = userService.getBaseMapper().selectList(null);
        WorkBookHelper workBookHelper = new WorkBookHelper();
        workBookHelper.exportWorkBook(response,list);
    } catch (IOException e) {
    }
}
```

## 使用

`http://localhost:8080/user/getExcel`

<img src="/assets/images/useage/02.gif"/>

生成的Excel文件内容：

<img src="/assets/images/useage/03.png"/>



# 1.LVGL 自带字体使用

**使用字体之前，应该在lv_conf.h中将对应的字体宏使能**
## 单个组件使用

```C
#1
void lv_style_set_text_font(lv_style_t * style, const lv_font_t * value)
#2
void lv_obj_set_style_text_font(lv_obj_t * obj, const lv_font_t * value)
```
通过设置格式，设置单个组件的字体，lv_font_t是在lvgl中定义的字体变量，通常格式为lv_font_name_size，例如你可以这样设置字体：
```C
static lv_obj_t *label1 = lv_label_create(btn1); //在btn1中添加标签
static lv_style_t style1; // 设置格式
lv_style_init(&style1); // 格式初始化
lv_style_set_text_font(&style1, &lv_font_montserrat_24); //设置格式字体
lv_obj_add_style(label1, &style1, LV_STATE_DEFAULT); //绑定组件

lv_obj_set_size(label1, LV_PCT(100), LV_PCT(80)); // 设置大小
lv_obj_align(label1, LV_ALIGN_CENTER, 0, 0); // 设置位置
lv_label_set_text(label1, "Hello World!"); // 设置内部文本
```
## 全局使用

也便是修改默认字体，在lv_conf.h中修改对应的宏
```C
#define LV_FONT_DEFAULT &lv_font_montserrat_14
```

# 2.自己导入字体并使用

## 制作LVGL字体文件

使用【在线字体转换器】(https://lvgl.io/tools/fontconverter)

![[Pasted image 20240612163156.png]]

## 声明字体
在将文件加入工程之后，还需要对字体进行声明，LVGL的自带的字体，在使能对应的宏之后，会完成对应自己的使能，我们自己定义的字体需要自己进行声明，在lv_conf.h中定义了宏LV_FONT_CUSTOM_DECLARE，我们将声明添加在这之后，他则会帮我们声明
```C
LV_FONT_CUSTOM_DECLARE   LV_FONT_DECLARE(my_font_1) LV_FONT_DECLARE(my_font_2)
```
也可以在使用的时候再声明
```C
LV_FONT_DECLARE(my_font)
```


**加入字库后，必须让工程代码处于UTF-8编码模式下，否则文字显示会出问题**
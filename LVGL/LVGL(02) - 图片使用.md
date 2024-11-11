LVGL默认支持PNG,BMP,JPG,SJPG和GIF动图等格式的图片显示。
# 存储形式

图像存储在位置
- 内部存储器（RAM或ROM）中的缓存空间或数组变量；
- 文件系统中的图片文件；

内部存储器中的图片以lv_img_dsc_t为形式存在，一张图片想要放在内部存储器中需要预先通过[工具](https://lvgl.io/tools/imageconverter)转换成lv_img_dsc_t的形式。

使用文件系统中的图片文件需要使用**LVGL的文件系统模块**。存储为文件的图像不会链接到生成的可执行文件中，并且必须在绘制之前读取到 RAM。因此，它们不像可 变图像那样资源友好。但是，它们更容易替换而无需重新编译主程序
# 颜色格式

lvgl支持多种内置颜色格式：
- LV_IMG_CF_TRUE_COLOR 只需存储RGB颜色（使用LVGL配置的任何颜色深度）。
- LV_IMG_CF_TRUE_COLOR_ALPHA 与LV_IMG_CF_TRUE_COLOR类似，但它还会为每个像素添加一个alpha（透明度）字节。
- LV_IMG_CF_TRUE_COLOR_CHROMA_KEYED 与LV_IMG_CF_TRUE_COLOR类似，但是如果像素具有LV_COLOR_TRANSP（在lv_conf.h中设置）颜色，则该像素将是透明的。
- LV_IMG_CF_INDEXED_1/2/4/8BIT 使用2、4、16或256色调色板，并以1、2、4或8位存储每个像素。
- LV_IMG_CF_ALPHA_1/2/4/8BIT 仅将Alpha值存储在1、2、4或8位上。 像素采用style.image.color和设置的不透明度的颜色。源图像必须是Alpha通道。这对于类似于字体的位图是理想的（整个图像是一种颜色，但是您希望能够更改它）

LV_IMG_CF_TRUE_COLOR图像的字节按以下顺序存储。
- 对于32位色深：
	Byte 0: 蓝色(Blue)
	Byte 1: 绿色(Green)
	Byte 2: 红色(Red)
	Byte 3: 透明度(Alpha)
- 对于16位色深：
	Byte 0: 绿色(Green )3位低位，蓝色(Blue)5位
	Byte 1: 红色(Red)5位，绿色(Green )3位
	Byte 2: 透明度(Alpha)字节（仅适用于LV_IMG_CF_TRUE_COLOR_ALPHA）
- 对于8位色深：
	Byte 0: Red 3 bit, Green 3 bit, Blue 2 bit
	Byte 2: Alpha byte (only with LV_IMG_CF_TRUE_COLOR_ALPHA)

也可以以 Raw 格式存储图像，以表明它没有使用其中一种内置颜色格式进行编码，并且需要使用**外部图像解码器**来解码图像。 
- LV_IMG_CF_RAW 表示基本的原始图像（例如 PNG 或 JPG 图像）。
- LV_IMG_CF_RAW_ALPHA 表示图像具有 alpha 并且为每个像素添加一个 alpha 字节。
- LV_IMG_CF_RAW_CHROME_KEYED 与“LV_IMG_CF_TRUE_COLOR_CHROMA_KEYED”中类似。
# 图像使用

## 显示TF卡图像文件
在完成LVGL的文件系统设置之后，便可以从文件系统中直接读取图像并显示：
```C
// 图片文件储存在TF卡目录/images/1.jpg
lv_obj_t *img= lv_img_create(lv_scr_act()); // 创建img对象
lv_img_set_src(img, "S:/images/1.jpg"); // 设置图像数据源，支持jpg, sjpg, png, bmp
lv_obj_align(obj, LV_ALIGN_CENTER, 0, 0);  // 居中显示
```
## 显示图像符号
除了 ASCII 范围，以下符号也从 FontAwesome 字体添加到内置字体中，LVGL中定义了一些常见的符号，添加到FontAwesome内置字体中

```C
lv_obj_t *obj = lv_img_create(lv_scr_act());
lv_img_set_src(obj, LV_SYMBOL_OK);
```
可以用来显示如：从摄像头读取到的图像数据，从网络上获取的图像数据等等

## 显示内部存储器中的图像
在内部存储器(ROM or RAM)中的图像都需要转化成lv_img_dsc_t的格式才能正常显示

```C
uint8_t my_img_data[] = {0x00, 0x01, 0x02, ...}; 
static lv_img_dsc_t my_img_dsc = { 
	.header.always_zero = 0, //始终为0
	.header.w = 80, //像素宽度(<=2048)
	.header.h = 60, //像素高度(<=2048)
	.data_size = 80 * 60 * LV_COLOR_DEPTH / 8, //data的长度
	.header.cf = LV_IMG_CF_TRUE_COLOR, // 设置颜色格式
	.data = my_img_data, //图像数组
};
```

在使用图像的时候类似与字体，需要先进行图像的声明才能够使用
```C
LV_IMG_DECLARE(image_girl_28x32);//声明图像
lv_obj_t * img = lv_img_create(lv_scr_act());//创建组件
lv_img_set_src(img, &image_girl_28x32);//设置图像源
```

## 显示GIF动画
与从外部图像读取一致，只是相关函数不一样
```C
// GIF动画
lv_obj_t *gif = lv_gif_create(lv_scr_act());
lv_gif_set_src(gif, "S:/images/1.gif");

lv_gif_restart(gif); // 重新播放GIF
```
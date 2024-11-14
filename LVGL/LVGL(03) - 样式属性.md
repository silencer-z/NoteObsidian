# 大小与位置属性
该样式属性与对象的宽度、高度、位置、对齐方式和布局相关。

|     名称     |  用途  |     值      |       默认值       |  继承  | 布局  |
| :--------: | :--: | :--------: | :-------------: | :--: | :-: |
|   width    | 对象宽度 | lv_coord_t | LV_SIZE_CONTENT | True |  \  |
| min_width  | 最小宽度 | lv_coord_t |        0        | True |  \  |
| max_width  | 最大宽度 | lv_coord_t |  LV_COORD_MAX   | True |  \  |
|   height   | 对象高度 | lv_coord_t | LV_SIZE_CONTENT | True |  \  |
| min_height | 最小高度 | lv_coord_t |        0        | True |  \  |
| max_height | 最大高度 | lv_coord_t |  LV_COORD_MAX   | True |  \  |
|     x      | X坐标  | lv_coord_t |        0        | True |  \  |
|     y      | Y坐标  | lv_coord_t |        0        | True |  \  |


## 对齐(Align)

## 
# 填充与边距属性
填充(Padding)可在边缘的内侧设置空间。 边距(margin )在边缘的外侧设置空间。意思是“我想要我周围的空间”。如果启用了布局或 自动调整，则这些属性通常由 Container 对象使用。但是，其他小部件也使用它们来设置间距。有关详细信息，请参见小部件的文档。

>[!NOTE] Boxing model
![[Pasted image 20241111154051.png]]
 **边界框**(Width&Height)：元素的宽度/高度
 **边框宽度**(Border)：边框的宽度
 **内边距**(Padding)：对象与其子元素之间的间距
 **外边距**(Outline)：对象外部的间距
 **内容**(Content)：内容区域，即边界框减去边框宽度和内边距的大小
 
- `pad_top`（lv_coord_t）在顶部设置填充。默认值：0。
- `pad_bottom`（lv_coord_t）在底部设置填充。默认值：0。
- `pad_left`（lv_coord_t）在左侧设置填充。默认值：0。
- `pad_right`（lv_coord_t）在右侧设置填充。默认值：0。
- `pad_row`（lv_coord_t）在行之间设置填充。默认值：0。
- `pad_column`（lv_coord_t）在列之间设置填充。默认值：0。

- `margin_top`（lv_coord_t）在顶部设置边距。默认值：0。
- `margin_bottom`（lv_coord_t）在底部设置边距。默认值：0。
- `margin_left`（lv_coord_t）在左边设置边距。默认值：0。
- `margin_right`（lv_coord_t）在右边设置边距。默认值：0。

# 背景属性
背景是一个可以具有渐变和圆角舍入简单矩形。

- `bg_color`（lv_color_t）指定背景的颜色。默认值：LV_COLOR_WHITE。
- `bg_opa`（lv_opa_t）指定背景的不透明度，0-255或者LV_OPA_0/TRANSP/COVER。默认值：LV_OPA_TRANSP。

- `bg_grad_color`（lv_color_t）指定背景渐变的颜色。`grad_dir` != LV_GRAD_DIR_NONE起效。默认值：LV_COLOR_WHITE
- `bg_grad_dir`（lv_grad_dir_t）指定渐变的方向。可以 LV_GRAD_DIR_NONE/HOR/VER。默认值：LV_GRAD_DIR_NONE。
- `bg_main_stop`（lv_coord_t）指定渐变应从何处开始。0：最左/最上位置，255：最右/最下位置。默认值：0。
- `bg_grad_stop`（lv_coord_t）指定渐变应在何处停止。0：最左/最上位置，255：最右/最下位置。预设值：255。
- `bg_main_opa`（lv_coord_t）指定渐变第一个颜色的透明度。默认值：255。
- `bg_grad_opa`（lv_coord_t）指定渐变第二个颜色透明度。默认值：255。
- `bg_grad`（lv_grad_dsc_t）设置渐变，将`grad_color`, `grad_dir`, `main_stop`, `grad_stop`包装。默认值：NULL。
- `bg_dither_mode`（lv_dither_mode_t）：设置渲染模式。LV_DITHER_NONE/ ORDERED/ ERR_DIFF 。默认值：LV_DITHER_NONE。

- `bg_image_src`（void \*）设置背景图片，可以是指向lv_image_dsc_t的指针、文件路径或LV_SYMBOL_。默认值：NULL。
- `bg_image_opa`（lv_opa_t）设置背景图片透明度，0-255或者LV_OPA_0/TRANSP/COVER。默认值：LV_OPA_COVER。
- `bg_image_recolor`（lv_color_t）设置要混合到背景图像的颜色。默认值：LV_COLOR_WHITE
- `bg_image_recolor_opa`（lv_opa_t）设置背景图像的亮度，0-255或者LV_OPA_0-100/ TRANSP/ COVER。默认值：LV_OPA_TRANSP。
- `bg_image_tiled`（bool）如果启用，背景图像将平铺。默认值：False。

# 边框属性
边框绘制在背景上方，也具有圆角舍入。

- `border_color`（lv_color_t）指定边框的颜色。默认值：LV_COLOR_BLACK。
- `border_opa`（lv_opa_t）指定边框的不透明度。默认值：LV_OPA_COVER。
- `border_width`（lv_coord_t）设置边框的宽度。默认值：0。
- `border_side`（lv_border_side_t）指定要绘制边框的哪一侧。LV_BORDER_SIDE_NONE/LEFT/ RIGHT/TOP/BOTTOM/FULL。OR-ed 值也是可能的。默认值：LV_BORDER_SIDE_FULL。
- `border_post`（bool）如果 true 在绘制完所有子级之后绘制边框。默认值：false。
# 轮廓属性
轮廓类似于边框，但绘制在对象外部。

- `outline_color`（lv_color_t）指定轮廓的颜色。默认值：LV_COLOR_BLACK。
- `outline_opa`（lv_opa_t）指定轮廓的不透明度。默认值：LV_OPA_COVER。
- `outline_width`（lv_coord_t）设置轮廓的宽度。默认值：0。
- `outline_pad`（lv_coord_t）设置对象和轮廓之间的空间。默认值：0。
# 阴影属性
阴影是对象下方的模糊区域。

- `shadow_color`（lv_color_t）指定阴影的颜色。默认值：LV_COLOR_BLACK。
- `shadow_opa`（lv_opa_t）指定阴影的不透明度。默认值：LV_OPA_TRANSP。
- `shadow_width`（lv_coord_t）设置轮廓的宽度（模糊大小） 。默认值：0。
- `shadow_ofs_x`（lv_coord_t）设置阴影的 X 偏移量。默认值：0。
- `shadow_ofs_y`（lv_coord_t）设置阴影的 Y 偏移量。默认值：0。
- `shadow_spread`（lv_coord_t）在每个方向上使阴影大于背景的值达到此值。默认值：0。
# 图片属性
图像的属性。

- `image_recolor`（lv_color_t）将此颜色混合到图案图像中。如果是符号（文本） ，它将是文本颜色。默认值：LV_COLOR_BLACK
- `image_recolor_opa`（lv_opa_t）重新着色的强度。LV_OPA_TRANSP 不重新着色 。默认值：LV_OPA_TRANSP
- `image_opa`（lv_opa_t）图像的不透明度。默认值：LV_OPA_COVER。
# 线条属性
线的属性

- `line_color`（lv_color_t）线条的颜色。默认值：LV_COLOR_BLACK
- `line_opa`（lv_opa_t）直线的不透明度。默认值：LV_OPA_COVER
- `line_width`（lv_coord_t）线的宽度。默认值：0。
- `line_dash_width`（lv_coord_t）破折号的宽度。仅对水平或垂直线绘制虚线。默认值：0。
- `line_dash_gap`（lv_coord_t）两条虚线之间的间隙。仅对水平或垂直线绘制虚线。默认值：0。
- `line_rounded`（bool）使能会绘制圆角的线尾。默认值：false。
# 圆弧属性
圆弧的属性

- `arc_width`（lv_coord_t）圆弧的宽度，默认值：0。
- `arc_rounded`（bool）使能使圆弧端点变圆，默认值：False。
- `arc_color`（lv_color_t）指定圆弧的颜色，默认值：LV_COLOR_WHITE。
- `arc_opa`（lv_opa_t）指定圆弧的透明度，默认值：LV_OPA_TRANSP。
- `arc_image_src`（void \*）设置遮罩圆弧的图像，可以是指向lv_image_dsc_t的指针、文件路径或LV_SYMBOL_。默认值：NULL。
# 文本属性
文本对象的属性。

- `text_color`（lv_color_t）：文本的颜色。默认值：LV_COLOR_BLACK。
- `text_opa`（lv_opa_t）：文本的不透明度。默认值：LV_OPA_COVER。
- `text_font`（const lv_font_t \*）：指向文本字体的指针。默认值：NULL。
- `text_letter_space`（lv_coord_t）：文本的字母空间。默认值：0。
- `text_line_space`（lv_coord_t）：文本的行距。默认值：0。
- `text_decor`（lv_text_decor_t）：添加文字修饰。默认值：LV_TEXT_DECOR_NONE。
- `text_align`（lv_text_align_t）：设置如何对齐文本的行，它不会对齐对象本身，只会对齐对象内部的线条。默认值：LV_TEXT_ALIGN_ALIGN。

# 移动属性
部件移动的属性

- transform_width（lv_coord_t）使用此值使对象两侧变宽，可以是像素或者百分比，相对于对象本身宽度。默认值：0。
- transform_height（lv_coord_t）使用此值使对象两侧变高，可以是像素或者百分比，相对于对象本身宽度。默认值：0。
- transform_x（lv_coord_t）在X方向上移动，在布局、对齐和其他定位后应用。可以是像素或者百分比，相对于对象本身宽度。默认值：0。
- transform_y（lv_coord_t）在Y方向上移动，在布局、对齐和其他定位后应用。可以是像素或者百分比，相对于对象本身宽度。默认值：0。
- transform_zoom（lv_coord_t）缩放类似图像的对象，256(LV_IMG_ZOOM_NONE)为正常大小，128为一半，512为两倍。默认值：0。
- transform_angle（lv_coord_t）旋转对象的角度，单位是0.1°，450就是旋转45°。默认值：0。
- transform_pivot_x
- transform_pivot_y

# 其他属性


大小和位置的设置
| 名称 | 用途 | 值  | Default | Inherited | Layout | Ext.draw |

width	对象宽度	像素、百分比、LV_SIZE_CONTENT	Widget dependent	\	YES	\
min_width	最小宽度	像素、百分比	0	\	YES	\
max_width	最大宽度	像素、百分比	LV_COORD_MAX	\	YES	\
height	对象高度	像素、百分比、LV_SIZE_CONTENT	Widget dependent	\	YES	\
min_height	最小高度	像素、百分比	0	\	YES	\
max_height	最大高度	像素、百分比	LV_COORD_MAX	\	YES	\
x	X坐标	像素、百分比	0	\	YES	\
y	Y坐标	像素、百分比	0	\	YES	\
align	参照父级	单独解释	LV_ALIGN_DEFAULT	\	YES	\
transform_width	变宽	像素、百分比、lv_pct(x)	0	\	\	YES
transform_height	变高	像素、百分比、lv_pct(x)	0	\	\	YES
translate_x	X方向移动	像素、百分比、lv_pct(x)	0	\	YES	\
translate_y	Y方向移动	像素、百分比、lv_pct(x)	0	\	YES	\
transform_zoom	缩放	256(LV_IMG_ZOOM_NONE) 其余值按此变换	0	\	YES	YES
transform_angle	旋转	0。1表示一度	0	\	YES	YES
transform_pivot_x	设置X坐标	相对于对象的左上角	0	\	\	\
transform_pivot_y	设置Y坐标	相对于对象的左上角	0	\	\	\
aligh：确定应该从父级的位置对齐方式。
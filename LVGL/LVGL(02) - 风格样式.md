LVGL样式包括状态（如默认、按下、聚焦等）、部件（如背景、边框、轮廓等）以及各种属性。对象可以有多个样式，**优先级高的样式覆盖低优先级**的。状态变化时，可以使用过渡动画平滑切换样式。此外，LVGL还支持主题，提供默认外观并允许自定义扩展。其具有以下特点：

- 样式是一个 `lv_style_t` 可以保存属性的**变量**，用于设置对象的外观
- 并非必须指定所有属性。未指定的属性将使用**默认值**
- 可以将样式分配给对象以更改其外观，可以被**任意数量的对象**使用
- 样式可以级联，可以将多个样式分配给一个对象，并且每种样式可以具有不同的属性
- 以后添加的样式具有更高的优先级
- 如果未在对象中指定，则某些属性可以从父级继承
- 对象可以具有比“普通”样式更高优先级的局部样式
- lvgl 可将属性分配给给定的状态
- 当对象更改状态时可以应用过渡

# 对象的状态(States)
对于一个对象，LVGL将其划分成多个状态：
- `LV_STATE_DEFAULT` (0x00): Normal, released
- `LV_STATE_CHECKED` (0x01): Toggled or checked
- `LV_STATE_FOCUSED` (0x02): Focused via keypad or encoder or clicked via touchpad/mouse
- `LV_STATE_EDITED`  (0x04): Edit by an encoder
- `LV_STATE_HOVERED` (0x08): Hovered by mouse (not supported now)
- `LV_STATE_PRESSED` (0x10): Pressed
- `LV_STATE_DISABLED` (0x20): Disabled or inactive

状态是支持组合的，例如： LV_STATE_FOCUSED | LV_STATE_PRESSED便是同时指定对象被聚焦或者按压状态。可以在每种状态和状态组合中定义样式属性。

如果未在状态中定义属性，则将使用**最佳匹配状态**的属性。他会按照上述状态的**排列顺序**以此寻找属性设定，越下面的优先级越高，优先级最低的是`LV_STATE_DEFAULT` 状态的属性。˛如果即使对于默认状态也未设置该属性，则将**使用默认值**。

# 对象的部件(Parts)
对于一个对象，LVGL将其划分为不同的部件，一个对象由这些部件构成，每一个部件也可以设定不同的样式，LVGL中存在以下预定义部分：
 - `LV_PART_MAIN`：主体背景
 - `LV_PART_SCROLLBAR`：滚动条
 - `LV_PART_INDICATOR`：指示器
 - `LV_PART_KNOB`：像把手一样用来调节值
 - `LV_PART_SELECTED`：表示当前选择的选项或部分
 - `LV_PART_ITEMS`：如果控件具有多个类似元素时使用
 - `LV_PART_CURSOR`：标记特定位置，例如文本区域或图表的光标
 - `LV_PART_CUSTOM_FIRST`：可以从这里开始添加自定义部分标识符
 -  ... 
不同的控件(widget)有不同的部件组合，并**不是所有部件都具有**。

# 样式的设置
在 LVGL 中，设置样式属性的方法有两个
## 普通样式设置
普通样式设置的最明显特点是：**共用**。它类似于一个共用的样式套装，用户可以往里面添加所需要修改的样式内容，将这个样式套装应用到某个部件时，其所包含的样式内容将会被全部应用到该部件中。使用此方法，这可以使样式设置变得非常**高效**。

在设置普通样式时，用户需要先调用 lv_style_t 结构体，**定义**样式变量，然后**初始化**样式以及**设置各种样式属性**，最后即可为目标对象**添加样式**。样式是存储在`lv_style_t`变量中，这个变量不能为局部变量，必须是static/全局/动态申请的变量。其相关函数如下：
```C
/**  
 * Initialize a style 
 * @param style pointer to a style to initialize 
 **/
void lv_style_init(lv_style_t * style);  
  
/**  
 * Clear all properties from a style and free all allocated memories. 
 * @param style pointer to a style 
 **/
void lv_style_reset(lv_style_t * style);

/**  
 * 设置属性的函数模板
 * @param style     样式指针
 * @param value     对应属性的属性值
 **/
void lv_style_set_<property_name>(lv_style_t *style, <value>);

/**  
 * Add a style to an object. 
 * @param obj       pointer to an object 
 * @param style     pointer to a style to add 
 * @param selector  OR-ed value of parts and states to add style
 **/
void lv_obj_add_style(struct _lv_obj_t * obj, lv_style_t * style, 
					  lv_style_selector_t selector);  
  
/**  
 * Add a style to an object. 
 * @param obj       pointer to an object 
 * @param style     pointer to a style to remove. Can be NULL
 * @param selector  OR-ed values of states and a part to remove style
 **/
void lv_obj_remove_style(struct _lv_obj_t * obj, lv_style_t * style, 
						 lv_style_selector_t selector);  
  
/**  
 * Remove all styles from an object 
 * @param obj       pointer to an object 
 **/
static inline void lv_obj_remove_style_all(struct _lv_obj_t * obj)  
{  
    lv_obj_remove_style(obj, NULL, LV_PART_ANY | LV_STATE_ANY);  
}
```
## 本地样式设置
本地样式的特点是：**设置简单，针对性强**。当用户界面的对象样式有较大差异时，可以使用本地样式进行单独的设置，本地样式的设置非常简单，只需要直接将样式设置到某个部件上即可。其相关函数如下：
```C
/**  
 * 设置属性的函数模板
 * @param obj       对象指针
 * @param value     对应属性的属性值
 * @param selector  状态&组成部分选择器
 **/
void lv_obj_set_style_<property_name>(struct _lv_obj_t * obj, <value>, 
									  lv_style_selector_t selector);
```
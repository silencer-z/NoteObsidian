与LVGL的许多其他部分类似，设置坐标的概念受到了CSS的启发。有很多属性和样式的设置与CSS很相似。
- 支持最小宽度、最大宽度、最小高度、最大高度
- 有像素、百分比和“内容（content）”单位
- 使用Boxing model(盒子模型)
- 支持flexbox和grid布局的部分功能

**空间单位**
>像素(pixel)：简单地说就是一个以像素为单位的位置。整数总是指像素。
>
>百分比(percentage)：是对象自身或其父对象大小的百分比。`lv_pct(value)` 。
>
>LV_SIZE_CONTENT(auto)：根据子对象设置宽度和高度，相当于CSS中的`auto`。

# 位置
通常有两种表示位置的方法，绝对坐标和相对坐标。前者直接指定对象的坐标，而后者则参考另一个对象的位置与其对齐。注意后者是通过**改变原点实现的对齐**，所以在对齐后设置X，Y的坐标实际是设置对于对齐点的偏移。
## 绝对坐标设置

```C
/**  
 * Set the position of an object relative to the set alignment. 
 * @param obj       pointer to an object 
 * @param x         new x coordinate 
 * @param y         new y coordinate 
 **/
void lv_obj_set_pos(struct _lv_obj_t * obj, lv_coord_t x, lv_coord_t y);  
  
/**  
 * Set the x coordinate of an object 
 * @param obj       pointer to an object 
 * @param x         new x coordinate  
 **/
void lv_obj_set_x(struct _lv_obj_t * obj, lv_coord_t x);  
  
/**  
 * Set the y coordinate of an object 
 * @param obj       pointer to an object 
 * @param y         new y coordinate 
 **/
void lv_obj_set_y(struct _lv_obj_t * obj, lv_coord_t y);
```
## 相对坐标设置(对齐)

```C
/**  
 * Align an object to the center on its parent. 
 * @param obj       pointer to an object to align 
 **/
static inline void lv_obj_center(struct _lv_obj_t * obj)  
{  
    lv_obj_align(obj, LV_ALIGN_CENTER, 0, 0);  
}
/**  
 * Change the alignment of an object. 
 * @param obj       pointer to an object to align 
 * @param align     对齐方式('lv_align_t'enum)外部对齐不能使用
 **/
void lv_obj_set_align(struct _lv_obj_t * obj, lv_align_t align);  
  
/**  
 * Change the alignment of an object and set new coordinates. 
 * @param obj       pointer to an object to align 
 * @param align     对齐方式('lv_align_t' enum)外部对齐不能使用
 * @param x_ofs     对齐后X的偏移
 * @param y_ofs     对齐后Y的偏移
 **/
void lv_obj_align(struct _lv_obj_t * obj, lv_align_t align, 
				  lv_coord_t x_ofs, lv_coord_t y_ofs);  
  
/**  
 * Align an object to an other object. 
 * @param obj       需要对齐对象的指针
 * @param base      对齐的目标，缺省则为父类
 * @param align     对齐方式('lv_align_t' enum) 
 * @param x_ofs     对齐后X的偏移
 * @param y_ofs     对齐后Y的偏移
 * @note            如果‘ base ’的位置或大小发生变化，‘ obj ’需要再次手动对齐
 **/
void lv_obj_align_to(struct _lv_obj_t * obj, const struct _lv_obj_t * base,
					 lv_align_t align, lv_coord_t x_ofs,lv_coord_t y_ofs);
```

LVGL有两种对齐方法，**内部对齐**和**外部对齐**，前者是用于设置**对象和其父类对象**之间的位置关系。后者则使用在两个对象**没有父子关系**的情况下。

![[Pasted image 20241111160218.png]]

内部对齐的枚举：
- `LV_ALIGN_TOP_LEFT`
- `LV_ALIGN_TOP_MID`
- `LV_ALIGN_TOP_RIGHT`
- `LV_ALIGN_BOTTOM_LEFT`
- `LV_ALIGN_BOTTOM_MID
- `LV_ALIGN_BOTTOM_RIGHT`
- `LV_ALIGN_LEFT_MID`
- `LV_ALIGN_RIGHT_MID`
- `LV_ALIGN_CENTER`
外部对齐的枚举：
- `LV_ALIGN_OUT_TOP_LEFT`
- `LV_ALIGN_OUT_TOP_MID`
- `LV_ALIGN_OUT_TOP_RIGHT`
- `LV_ALIGN_OUT_BOTTOM_LEFT`
- `LV_ALIGN_OUT_BOTTOM_MID`
- `LV_ALIGN_OUT_BOTTOM_RIGHT`
- `LV_ALIGN_OUT_LEFT_MID`
- `LV_ALIGN_OUT_RIGHT_MID`

>[! WARNING] 所有对齐函数，在被对齐目标位置发生改变的时候，需要再设置一边对齐。
# 大小
LVGL参考CSS的设置，也采用盒子模型。具体盒子模型的设置我们在样式中讨论，这里只指定其宽度和高度。
```C
/**  
 * Set the width of an object 
 * @param obj       pointer to an object 
 * @param w         the new width
 **/
 void lv_obj_set_width(struct _lv_obj_t * obj, lv_coord_t w);  
  
/**  
 * Set the height of an object 
 * @param obj       pointer to an object 
 * @param h         the new height
 **/
void lv_obj_set_height(struct _lv_obj_t * obj, lv_coord_t h);
/**  
 * Set the size of an object. 
 * @param obj       pointer to an object 
 * @param w         the new width 
 * @param h         the new height 
 **/
void lv_obj_set_size(struct _lv_obj_t * obj, lv_coord_t w, lv_coord_t h);
```

在设置宽度和高度的时候其单位不光可以是整数(像素)，也可以是`lv_pct()`根据父类内容区域大小计算，其也支持特殊值`LV_SIZE_CONTENT`，这意味着对象在相应方向上的大小将被设置为其子对象恰好所需的大小
>[!NOTE] 使用`LV_SIZE_CONTENT`的时候只有右侧和底部的子对象会被考虑，顶部和左侧对象依然会被裁切。此外拥有`LV_OBJ_FLAG_HIDDEN`和`LV_OBJ_FLAG_FLOATING`的对象会被忽略

# 布局
布局可以更新对象子对象的位置和大小。它们可以用于自动排列子对象成一行或一列，或者以更复杂的形式排列，布局设置的位置和大小会**覆盖x、y、宽度和高度设置**。每个布局都有一个相同的函数： `lv_obj_set_layout()` 用于在对象上设置布局。
```C
/**  
 * Set a layout for an object 
 * @param obj       pointer to an object 
 * @param layout    pointer to a layout descriptor to set 
 **/
void lv_obj_set_layout(struct _lv_obj_t * obj, uint32_t layout);
```
同样的LVGL将CSS中`Flex`和`Grid`布局引入，但也不完全相同，具体的内容在布局中更详细描述。
 - **FlexBox**：将对象排列成行或列，支持换行和扩展项目
 - **Grid**：在二维表中将对象排列成固定位置
>[!NOTE] 在使用布局的时候一些标志位也会影响布局的行为：
> - `LV_OBJ_FLAG_HIDDEN`:隐藏的对象在布局计算中被忽略
> - `LV_OBJ_FLAG_IGNORE_LAYOUT`:该对象被布局简单地忽略。它的坐标可以像常规那样设置
> - `LV_OBJ_FLAG_FLOATING`:与`LV_OBJ_FLAG_IGNORE_LAYOUT`相同，但会被设置`LV_SIZE_CONTENT`计算时被忽略
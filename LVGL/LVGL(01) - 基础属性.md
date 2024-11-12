与LVGL的许多其他部分类似，设置坐标的概念受到了CSS的启发。有很多属性和样式的设置与CSS很相似。
- 支持最小宽度、最大宽度、最小高度、最大高度
- 有像素、百分比和“内容（content）”单位
- 使用Boxing model(盒子模型)
- 支持flexbox和grid布局的部分功能

**空间单位**
>像素(pixel)：简单地说就是一个以像素为单位的位置。整数总是指像素。

>百分比(percentage)：是对象自身或其父对象大小的百分比。`lv_pct(value)` 。

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

## Boxing model
![[Pasted image 20241111154051.png]]
 **边界框**(Width&Height)：元素的宽度/高度
 **边框宽度**(Border)：边框的宽度
 **内边距**(Padding)：对象与其子元素之间的间距
 **外边距**(Outline)：对象外部的间距
 **内容**(Content)：内容区域，即边界框减去边框宽度和内边距的大小

# 布局

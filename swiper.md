## swiper

+ 用swiper做了个移动端的选项卡，但是每个选项里的内容高度不一样，而swiper-wrapper是显示的最大高度，所以有些swiper-slide容器会有很大的空白 解决办法是在css添加如下代码:

```css
swiper-slide{height:10px}    swiper-slide-active { height:auto}
``````




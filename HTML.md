# Knowledge points about HTML

# 图片
使用图片时注意以下几点：

* 一定设置图片的width和height属性
  这两个属性可以引起图片的拉伸和压缩，但是不要这么做，要设置为真正图片的大小，因为压缩会导致带宽浪费，拉伸会影响图片效果，如果一定要这么做，在css中做。这里保留这两个属性是在图片加载缓慢时，为页面预留一定的大小，提升页面的性能和体验。
* 对于有意义的图片要使用img标签，如果是装饰使用，使用css background-image
* 为图片加caption使用figure
```HTML
  <figure>
    <img src="img.jpg">
    <figcaption>图2-1</figcaption>
  </figure>
```

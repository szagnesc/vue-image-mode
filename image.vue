<template>
  <div class="imgContainer">
    <slot></slot>
    <img ref="img" :src="lazyLoad===false?src:defaultSrc" :data-src="src"
         @error="imgError($event)"
         @load="imageLoaded($event)"/>
  </div>
</template>

<style scoped>
  .imgContainer {
    position: relative;
    text-align: center;
    overflow: hidden;
  }

  .imgContainer img {
    width:100%;
    height:100%;
    vertical-align: middle;
    /*z-index: 9;*/
  }

</style>
<script>
/*
  *@event {binderror, bindload}
  *@binderror 当错误发生时，发布到 AppService 的事件名，事件对象event.detail = {errMsg: 'something wrong'}
  *@bindload 当图片载入完毕时，发布到 AppService 的事件名，事件对象event.detail = {height:'图片高度px', width:'图片宽度px'}
  *
  *@param lazyLoad, src,mode
  * mode有13种模式（scaleToFill，aspectFit，aspectFill，widthFix，top,bottom,center,left,right,top left,top right,bottom left,bottom right）
  *
 */

export default {
  props: {
    //  图片懒加载。只针对page与scroll-view下的image有效
    lazyLoad: {
      type: Boolean,
      default: false
    },
    //  图片路径
    src: {
      type: String,
      default: ''
    },
    // 图片模式
    mode: {
      type: String,
      default: ''
    }
  },
  data () {
    return {
      defaultSrc: require('../../../static/images/lazyloadImg.png'),
      loadImg: false,
      imgList: Array,
      delay: null,
      flag: false
    }
  },
  methods: {
    //  图片加载成功之后设置模式，传递onload事件
    imageLoaded (e) {
      const imgObj = this.$refs.img
      const imgParent = this.$refs.img.parentNode
      const imgWidth = imgObj.width
      const imgHeight = imgObj.height
      const parentWidth = imgParent.clientWidth
      const parentHeight = imgParent.clientHeight
      switch (this.mode) {
        case 'scaleToFill':
          imgObj.style.width = '100%'
          imgObj.style.height = '100%'
          break
        case 'aspectFit':
          if (imgWidth / imgHeight > parentWidth / parentHeight) {
            imgObj.style.width = '100%'
            imgObj.style.height = 'auto'
            imgObj.style.position = 'absolute'
            imgObj.style.top = '50%'
            imgObj.style.left = 0
            imgObj.style.transform = 'translateY(-50%)'
          } else {
            imgObj.style.width = 'auto'
            imgObj.style.height = '100%'
            imgObj.style.position = 'absolute'
            imgObj.style.left = '50%'
            imgObj.style.top = 0
            imgObj.style.transform = 'translateX(-50%)'
          }
          break
        case 'aspectFill':
          if (imgWidth / imgHeight > parentWidth / parentHeight) {
            imgObj.style.width = 'auto'
            imgObj.style.height = '100%'
            imgObj.style.position = 'absolute'
            imgObj.style.left = '50%'
            imgObj.style.top = 0
            imgObj.style.transform = 'translateX(-50%)'
          } else {
            imgObj.style.width = '100%'
            imgObj.style.height = 'auto'
            imgObj.style.position = 'absolute'
            imgObj.style.top = '50%'
            imgObj.style.left = 0
            imgObj.style.transform = 'translateY(-50%)'
          }
          break
        case 'widthFix':
          imgObj.style.width = '100%'
          imgParent.style.height = 'auto'
          break
        case 'top':
          imgObj.style.width = 'auto'
          imgObj.style.height = 'auto'
          imgObj.style.verticalAlign = 'top'
          imgObj.style.position = 'absolute'
          imgObj.style.left = '50%'
          imgObj.style.transform = 'translateX(-50%)'
          break
        case 'bottom':
          imgObj.style.width = 'auto'
          imgObj.style.height = 'auto'
          imgObj.style.position = 'absolute'
          imgObj.style.left = '50%'
          imgObj.style.transform = 'translateX(-50%)'
          imgObj.style.bottom = '0'
          break
        case 'center':
          imgObj.style.width = 'auto'
          imgObj.style.height = 'auto'
          imgObj.style.position = 'absolute'
          imgObj.style.left = '50%'
          imgObj.style.top = '50%'
          imgObj.style.transform = 'translate(-50%,-50%)'
          break
        case 'left':
          imgObj.style.width = 'auto'
          imgObj.style.height = 'auto'
          imgObj.style.position = 'absolute'
          imgObj.style.top = '50%'
          imgObj.style.left = 0
          imgObj.style.transform = 'translateY(-50%)'
          break
        case 'right':
          imgObj.style.width = 'auto'
          imgObj.style.height = 'auto'
          imgObj.style.position = 'absolute'
          imgObj.style.top = '50%'
          imgObj.style.right = 0
          imgObj.style.transform = 'translateY(-50%)'
          break
        case 'top right':
          imgObj.style.width = 'auto'
          imgObj.style.height = 'auto'
          imgObj.style.position = 'absolute'
          imgObj.style.right = '0'
          imgObj.style.top = '0'
          break
        case 'top left':
          imgObj.style.width = 'auto'
          imgObj.style.height = 'auto'
          imgObj.style.position = 'absolute'
          imgObj.style.left = '0'
          imgObj.style.top = '0'
          break
        case 'bottom left':
          imgObj.style.width = 'auto'
          imgObj.style.height = 'auto'
          imgObj.style.position = 'absolute'
          imgObj.style.left = '0'
          imgObj.style.bottom = '0'
          break
        case 'bottom right':
          imgObj.style.width = 'auto'
          imgObj.style.height = 'auto'
          imgObj.style.position = 'absolute'
          imgObj.style.right = '0'
          imgObj.style.bottom = '0'
          break
        default:
          imgObj.style.width = '100%'
          imgObj.style.height = '100%'
      }
      this.loadImg = true
      this.$emit('bindload', {
        type: 'load',
        detail: {
          height: imgObj.height,
          width: imgObj.width
        }
      })
    },
    //  图片加载失败事件
    imgError (e) {
      this.$emit('binderror', {
        type: 'error',
        detail: {
          imgError: e.target.currentSrc + '出现错误'
        }
      })
    },
    //  防止短时间内多次触发
    scrollDelay () {
      clearTimeout(this.delay)
      this.delay = setTimeout(() => {
        if (this.flag === false) {
          this.imgLazyLoaded()
        }
      }, 250)
    },
    //  判断图片是否出现在可视范围内
    imgIsShow () {
      let coords = this.$refs.img.getBoundingClientRect()
      return coords.top <= document.documentElement.clientHeight
    },
    //  给图片赋值
    imgLazyLoaded () {
      if (this.imgIsShow()) {
        this.flag = true
        this.$refs.img.src = this.$refs.img.getAttribute('data-src')
        this.imageLoaded()
      } else {
        this.$refs.img.style.width = '100%'
        this.$refs.img.style.height = '100%'
      }
    }
  },
  mounted () {
    if (this.lazyLoad) {
      //  图片相对视口的位置
      this.scrollDelay()
      window.addEventListener('scroll', this.scrollDelay, false)
    }
  }
}
</script>

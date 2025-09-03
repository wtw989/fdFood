<template>
  <div class="page">
    <!-- 顶部搜索区域 -->
    <header class="search-bar">
      <button class="btn-category" type="button" @click="toggleCategory">
        分类 <img src="@/assets/images/下拉@2x.png" alt="下拉"/>
      </button>
      <div class="search-input">
        <input
          type="search"
          v-model="searchKeyword"
          placeholder="请输入关键词搜索"
          @keyup.enter="handleSearch"
        />
        <button class="btn-search" type="button" @click="handleSearch">
          <img src="@/assets/images/搜索@2x.png" alt="搜索"/>
        </button>
      </div>
    </header>

    <!-- 横幅轮播图 -->
    <section class="banner">
      <div class="banner-container">
        <div
          class="banner-slides"
          :style="{ transform: `translateX(-${currentSlide * 100}%)` }"
          @touchstart="handleTouchStart"
          @touchmove="handleTouchMove"
          @touchend="handleTouchEnd"
          @mousedown="handleMouseDown"
          @mousemove="handleMouseMove"
          @mouseup="handleMouseUp"
          @mouseleave="handleMouseUp">
          <div
            v-for="(slide, index) in bannerSlides"
            :key="index"
            class="banner-slide"
          >
            <img :src="slide.image" :alt="slide.alt"/>
          </div>
        </div>

        <div class="banner-dots">
          <div
            v-for="(dot, index) in bannerSlides"
            :key="index"
            :class="['banner-dot', { active: index === currentSlide }]"
            @click="goToSlide(index)"
          ></div>
        </div>
      </div>
    </section>

    <!-- 产品区域 -->
    <section class="section" v-for="(category, index) in productCategories" :key="index">
      <h2 class="section-title">{{ category.name }}</h2>
      <div class="cards" v-if="category.layout === 'cards'">
        <div class="two-col">
          <a
            v-for="(product, productIndex) in category.products"
            :key="productIndex"
            :href="product.link"
            class="card"
          >
            <img :src="product.image" :alt="product.name"/>
          </a>
        </div>
        <div v-if="category.caption" class="caption">{{ category.caption }}</div>
      </div>
      <div class="grid" v-if="category.layout === 'grid'">
        <div
          v-for="(product, productIndex) in category.products"
          :key="productIndex"
          class="good"
          :class="category.showCTA ? 'squareBg' : ''"
        >
          <img :src="product.image" :alt="product.name"/>
          <div class="pill" v-if="product.caption">{{ product.caption }}</div>
        </div>
      </div>
      <div class="pill"  v-if="category.showCTA">{{ category.name }}</div>
    </section>
  </div>
</template>

<script>
export default {
  name: 'ProductHome',
  data() {
    return {
      searchKeyword: '',
      currentSlide: 0,
      autoSlideInterval: null,
      bannerSlides: [
        {
          image: require('@/assets/images/banner@2x.png'),
          alt: '促销横幅1'
        },
        {
          image: require('@/assets/images/banner@2x.png'),
          alt: '促销横幅2'
        },
        {
          image: require('@/assets/images/banner@2x.png'),
          alt: '促销横幅3'
        }
      ],
      productCategories: [
        {
          name: '恒都牛肉',
          layout: 'cards',
          showCTA: false,
          caption: '恒都京东自营旗舰店',
          products: [
            {
              image: require('@/assets/images/test.png'),
              name: '牛肉1',
              link: '#'
            },
            {
              image: require('@/assets/images/test.png'),
              name: '牛肉2',
              link: '#'
            }
          ]
        },
        {
          name: '丰都麻辣鸡',
          layout: 'grid',
          showCTA: false,
          products: [
            {
              image: require('@/assets/images/test.png'),
              name: '麻辣鸡1',
              caption: '叁石哥丰都麻辣鸡'
            },
            {
              image: require('@/assets/images/test.png'),
              name: '麻辣鸡2',
              caption: '源小幺丰都麻辣鸡工坊'
            },
            {
              image: require('@/assets/images/test.png'),
              name: '麻辣鸡3',
              caption: '抓卤记丰都麻辣鸡'
            },
            {
              image: require('@/assets/images/test.png'),
              name: '麻辣鸡4',
              caption: '华明孙记食品旗舰店'
            }
          ]
        },
        {
          name: '乡茹榨菜',
          layout: 'grid',
          showCTA: true,
          products: [
            {
              image: require('@/assets/images/test.png'),
              name: '榨菜1',
              caption: '',
              link: '#'
            },
            {
              image: require('@/assets/images/test.png'),
              name: '榨菜2',
              caption: '',
              link: '#'
            }
          ]
        }
      ],
      // 触摸/鼠标滑动相关数据
      startX: 0,
      startY: 0,
      isDragging: false,
      dragOffset: 0
    }
  },
  mounted() {
    this.startAutoSlide();
  },
  beforeDestroy() {
    this.stopAutoSlide();
  },
  methods: {
    // 搜索功能
    handleSearch() {
      if (this.searchKeyword.trim()) {
        console.log('搜索关键词:', this.searchKeyword);
        // 这里可以添加实际的搜索逻辑
      }
    },

    // 切换分类
    toggleCategory() {
      console.log('切换分类');
      // 这里可以添加分类切换逻辑
    },

    // 轮播图相关方法
    goToSlide(index) {
      this.currentSlide = index;
      this.resetAutoSlide();
    },

    nextSlide() {
      this.currentSlide = (this.currentSlide + 1) % this.bannerSlides.length;
    },

    prevSlide() {
      this.currentSlide = (this.currentSlide - 1 + this.bannerSlides.length) % this.bannerSlides.length;
    },

    startAutoSlide() {
      this.autoSlideInterval = setInterval(() => {
        this.nextSlide();
      }, 3000);
    },

    stopAutoSlide() {
      if (this.autoSlideInterval) {
        clearInterval(this.autoSlideInterval);
        this.autoSlideInterval = null;
      }
    },

    resetAutoSlide() {
      this.stopAutoSlide();
      this.startAutoSlide();
    },

    // 触摸事件处理
    handleTouchStart(e) {
      this.isDragging = true;
      this.startX = e.touches[0].clientX;
      this.startY = e.touches[0].clientY;
      this.stopAutoSlide();
    },

    handleTouchMove(e) {
      if (!this.isDragging) return;

      const currentX = e.touches[0].clientX;
      const currentY = e.touches[0].clientY;

      // 计算滑动距离
      const diffX = currentX - this.startX;
      const diffY = currentY - this.startY;

      // 如果垂直滑动距离大于水平滑动距离，则不处理滑动
      if (Math.abs(diffY) > Math.abs(diffX)) {
        return;
      }

      e.preventDefault();
      this.dragOffset = diffX;
    },

    handleTouchEnd() {
      if (!this.isDragging) return;

      const threshold = window.innerWidth / 4; // 滑动阈值为屏幕宽度的1/4

      if (this.dragOffset > threshold) {
        this.prevSlide();
      } else if (this.dragOffset < -threshold) {
        this.nextSlide();
      }

      this.isDragging = false;
      this.dragOffset = 0;
      this.startAutoSlide();
    },

    // 鼠标事件处理
    handleMouseDown(e) {
      this.isDragging = true;
      this.startX = e.clientX;
      this.startY = e.clientY;
      this.stopAutoSlide();
    },

    handleMouseMove(e) {
      if (!this.isDragging) return;

      const currentX = e.clientX;
      const currentY = e.clientY;

      // 计算滑动距离
      const diffX = currentX - this.startX;
      const diffY = currentY - this.startY;

      // 如果垂直滑动距离大于水平滑动距离，则不处理滑动
      if (Math.abs(diffY) > Math.abs(diffX)) {
        return;
      }

      e.preventDefault();
      this.dragOffset = diffX;
    },

    handleMouseUp() {
      if (!this.isDragging) return;

      const threshold = window.innerWidth / 4; // 滑动阈值为屏幕宽度的1/4

      if (this.dragOffset > threshold) {
        this.prevSlide();
      } else if (this.dragOffset < -threshold) {
        this.nextSlide();
      }

      this.isDragging = false;
      this.dragOffset = 0;
      this.startAutoSlide();
    }
  }
}
</script>

<style scoped>
*, *::before, *::after {
  box-sizing: border-box;
}

.page {
  max-width: 750px;
  margin: 0 auto;
  padding-bottom: 24px;
  background: #fff;
  color: #1f2d3d;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans", "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", sans-serif;
}

/* 顶部搜索区域 */
.search-bar {
  position: sticky;
  top: 0;
  z-index: 10;
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px;
  background: #fff;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.04);
}

.btn-category {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 8px 12px;
  background: #fff;
  color: #757575;
  border: 1px solid #539ED3;
  border-radius: 20px;
  font-size: 14px;
  white-space: nowrap;
  cursor: pointer;
  transition: background-color 0.3s;
}

.btn-category:hover {
  background: #e9f5ff;
}

.btn-category img {
  width: 16px;
  height: 16px;
  object-fit: contain;
}

.search-input {
  flex: 1;
  display: flex;
  align-items: center;
  border: 1px solid #539ED3;
  border-radius: 24px;
  overflow: hidden;
  background: #fff;
}

.search-input input {
  flex: 1;
  border: 0;
  outline: none;
  padding: 10px 12px;
  font-size: 14px;
}

.btn-search {
  width: 54px;
  height: 36px;
  border: 0;
  border-radius: 18px;
  background: #4396CF;
  color: #fff;
  font-size: 16px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: background-color 0.3s;
  margin: 1px;
}

.btn-search:hover {
  background: #3a7bb8;
}

.btn-search img {
  width: 20px;
  height: 20px;
  object-fit: contain;
}

/* 横幅 */
.banner {
  padding: 12px;
  position: relative;
  overflow: hidden;
}

.banner-container {
  position: relative;
  width: 100%;
  height: 0;
  padding-bottom: 40%;
  overflow: hidden
}

.banner-slides {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  transition: transform 0.5s ease-in-out;
}

.banner-slide {
  min-width: 100%;
  height: 100%;
  flex-shrink: 0;
}

.banner-slide img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: 16px;
  display: block;
}

.banner-dots {
  position: absolute;
  bottom: 10px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 8px;
  z-index: 2;
}

.banner-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.5);
  cursor: pointer;
  transition: background 0.3s;
}

.banner-dot.active {
  background: #fff;
}

/* 分区标题 */
.section {
  padding: 0 12px;
  margin: 20px 0;
}

.section-title {
  margin: 8px 0 12px;
  padding: 10px 16px;
  text-align: center;
  color: #fff;
  font-weight: 700;
  letter-spacing: 2px;
  border-radius: 10px;
  background: linear-gradient(90deg, #ACDDEF 0%, #4397D0 50%, #ACDDEF 100%);
  box-shadow: 0 6px 14px rgba(93, 183, 255, 0.25);
  font-size: 20px;
}

/* 卡片与网格 */
.cards {
  background: url("../../assets/images/背景@2x.png")  0 0 / 100% 100% no-repeat;
  padding: 14px;
}

.cards .caption{
  text-align: center;
  padding: 10px 10px 0;
  font-size: 14px;
}

.two-col {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.card {
  display: block;
  background: #fff;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
  text-decoration: none;
  color: inherit;
  transition: transform 0.3s, box-shadow 0.3s;
}

.card:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.1);
}

.card img {
  width: 100%;
  height: 160px;
  object-fit: cover;
  display: block;
}

.card .caption {
  padding: 10px 8px 12px;
  text-align: center;
  color: #666;
  font-size: 13px;
}

/* 2x2 商品宫格 */
.grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px 18px;
}

.grid .good {
  background: url("../../assets/images/背景@2x(1).png")  0 0 / 100% 100% no-repeat;
  padding: 10px;
}

.grid .squareBg{
  background: url("../../assets/images/背景@2x(2).png")  0 0 / 100% 100% no-repeat;
}

.grid .good img {
  width: 100%;
  height: 150px;
  object-fit: cover;
  border-radius: 10px;
  display: block;
}

.pill {
  margin: 10px auto 0;
  padding: 8px 14px;
  color: #fff;
  font-size: 13px;
  text-align: center;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  background: url("../../assets/images/标题背景@2x.png") center no-repeat;
  background-size: contain;
}


/* 小屏优化 */
@media (max-width: 360px) {
  .card img {
    height: 140px;
  }

  .grid .good img {
    height: 130px;
  }
}

/* 响应式设计 */
@media (max-width: 480px) {
  .search-bar {
    padding: 8px;
    gap: 8px;
  }

  .btn-category {
    padding: 8px 10px;
    font-size: 13px;
  }

  .search-input input {
    padding: 8px 10px;
    font-size: 13px;
  }

  .btn-search {
    width: 48px;
    height: 32px;
  }

  .section-title {
    font-size: 18px;
    padding: 8px 14px;
  }
}
</style>


# Trillo

[Demo](https://wtlin1228.github.io/trillo/)

### DEV mode

`$ npm start`

### PRODUCTION mode

`$ npm build:css`

### 如何使用 SVG

1. 將所有的 SVG 用在同一個檔案，像是 `img/sprite.svg`，這樣子就只需要一次 request 就可以拿到所有的 icon
2. 在 HTML 中引入想要的 SVG，只要在 `img/sprite.svg` 後面加上 `#icon-name` 就可以了

```HTML
<svg class="user-nav__icon">
  <use xlink:href="img/sprite.svg#icon-bookmark"></use>
</svg>
```

3. 這個做法的好處是可以直接在 CSS 中對 SVG 進行上色

```CSS
&__icon {
  /* 指定大小 */
  height: 2.25rem;
  width: 2.25rem;
  /* 指定顏色 */
  fill: var(--color-grey-dark-2);
}
```

### 製作 Sidebar 動畫

1. HTML 的部分

```HTML
<li class="side-nav__item">
  <a href="#" class="side-nav__link">
    <svg class="side-nav__icon">
      <use xlink:href="img/sprite.svg#icon-aircraft-take-off"></use>
    </svg>
    <span>Flight</span>
  </a>
</li>
```

2. CSS 的部分

```CSS
.side-nav {
  font-size: 1.4rem;
  list-style: none;
  margin-top: 3.5rem;

  &__item {
    position: relative;

    &:not(:last-child) {
      margin-bottom: .5rem;
    }
  }

  /*  
  *   幫 item 加上 before，然後使用絕對位置，
  *   參考 item 本身，這樣只要當 height/width = 100%/100% 的時候就會剛好覆蓋整個 item ，
  *   一開始的 transform: scaleY(0)，就可以把 before 藏起來。
  *
  *   注意這邊的 transition，可以針對不同的 attribute 做設定，
  *   讓 width 延遲 .2s 是為了先讓 height 變成 100% 之後再往右邊長出來。
  *   時間函數可以參考 https://cubic-bezier.com/#.17,.67,.83,.67
  */
  &__item:before {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    width: 3px;
    background-color: var(--color-primary);
    transform: scaleY(0);
    transition: transform .2s, 
                width .4s cubic-bezier(1, 0, 0, 1) .2s,
                background-color .1s;
  }

  /* 
  *  如果 hover 發生或是這個 item 是 active 的狀態，
  *  將 width 變成 100%，transform: scaleY(1)，就可以覆蓋整個 item，
  */
  &__item:hover::before,
  &__item--active::before {
    transform: scaleY(1);
    width: 100%;
  }

  &__item:active::before {
    background-color: var(--color-primary-light);
  }

  /* 注意 z-index 一定要有 position 的屬性才會有用 */
  &__link:link,
  &__link:visited {
    color: var(--color-grey-light-1);
    text-decoration: none;
    text-transform: uppercase;
    display: block;
    padding: 1.5rem 3rem;
    position: relative;
    z-index: 10;

    display: flex;
    align-items: center;
  }

  /* currentColor = parent 或是 自己的顏色 */
  &__icon {
    width: 1.75rem;
    height: 1.75rem;
    margin-right: 2rem;
    fill: currentColor;
  }
}
```

### 在 flexbox 中使用 margin: auto

當我們想要排舞個東西的時候，如果想要排成 1 2 3       4 5，那麼我們可以直接在第三個元素加上 `flex: 1`，如此一來第三個元素就會自動被拉長，可是我們不想要元素被拉長，此時可以寫 `margin-right: auto` 就可以達成一樣的效果，然後元素也不會被拉長

### 利用 flexbox 來將 inline block 置中

可以直接將 `<div>` 設成 `display: flex` 就不會有多餘的上下空白存在，利用這個方法可以取代 `font-size: 0; line-height: 0`

```HTML
<div>
  <svg />
  <svg />
  <svg />
  <svg />
  <svg />
</div>
```

### 設計 button 動畫

在想要有動畫的元件加上 `animation: pulsate 1s infinite;` 就可以讓這個動畫無限重播

```CSS
@keyframes pulsate {
  0% {
    transform: scale(1);  
    box-shadow: none
  }

  50% {
    transform: scale(1.05);
    box-shadow: 0 1rem 4rem rgba(0,0,0,.25);
  }

  100% {
    transform: scale(1);  
    box-shadow: none
  }
}
```

### 可以將元素的高度延展到跟 parent 一樣長

在 flexbox 下，使用 `align-self: stretch;` 就可以達成

### 使用 mask 直接從 CSS 引入 SVG

可以先給個顏色，然後用 `-webkit-mask-image` 來蓋著它

```CSS
&__item::before {
  content: "";
  display: inline-block;
  height: 1rem;
  width: 1rem;
  margin-right: .7rem;

  // Newer browsers - masks
  background-color: var(--color-primary);
  -webkit-mask-image: url(../img/chevron-thin-right.svg);
  mask-image: url(../img/chevron-thin-right.svg);
  -webkit-mask-size: cover;
  mask-size: cover;
}
```

### 不同瀏覽器相容性問題

可以用 `@supports` 來解決

但是如果這樣寫的話，`background-image`也會影響到有 mask 的狀況，所以記得要把 `background-image: none;`

```CSS
/* Older browsers */
background-image: url(../img/chevron-thin-right.svg);
background-size: cover;

/* Newer browsers - masks */
@supports (-webkit-mask-image: url()) or (mask-image: url()) {
  background-color: var(--color-primary);
  -webkit-mask-image: url(../img/chevron-thin-right.svg);
  mask-image: url(../img/chevron-thin-right.svg);
  -webkit-mask-size: cover;
  mask-size: cover;

  /* 需要加上這個 */
  background-image: none;
}
```

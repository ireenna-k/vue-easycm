<template>
  <div class="cm-container" :style="axisComputed" v-if="show" @mouseover=handleMouseover($event) @keydown="handleKeyDown($event)" ref="contextMenuContainer">
    <svg aria-hidden="true" style="position: absolute; width: 0px; height: 0px; overflow: hidden;"><symbol id="icon-youjiantou" viewBox="0 0 1024 1024"><path d="M288.791335 65.582671l446.41733 446.417329-446.41733 446.417329z"></path></symbol></svg>
    <!--first-->
    <ul class="cm-ul cm-ul-1 easy-cm-ul"
        :class="underline?'cm-underline':''">
      <li v-for="(item, index) in list" :style="liStyle">
        <div @click.stop="callback([index])"
             :class="firstLeft?'cm-left':''"  tabindex="0">
          <i :class="item.icon"></i>
          <span v-if="item.shortcut" class="shortcut">{{item.shortcut}}</span>
          <span>{{item.text}}</span>
          <svg class="icon" aria-hidden="true"
               v-if="arrow && item.children && item.children.length > 0">
              <use xlink:href="#icon-youjiantou"></use>
          </svg>
        </div>
        <!--second-->
        <ul class="cm-ul cm-ul-2 easy-cm-ul"
            :style="secondBorderCheck(index)"
            :class="underline?'cm-underline':''"
            v-if="item.children && item.children.length > 0" >
          <li v-for="(second, si) in item.children"
              :style="liStyle">
            <div @click.stop="callback([index,si])"
                 :class="secondLeft?'cm-left':''" tabindex="0">
              <i :class="second.icon"></i>
              <span v-if="second.shortcut" class="sub-shortcut">{{second.shortcut}}</span>
              <span>{{second.text}}</span>
              <svg class="icon" aria-hidden="true"
                   v-if="arrow && second.children && second.children.length > 0">
                  <use xlink:href="#icon-youjiantou"></use>
              </svg>
            </div>
            <!--third-->
            <ul class="cm-ul cm-ul-3 easy-cm-ul"
                :style="thirdBorderCheck(index,si)"
                :class="underline?'cm-underline':''"
                v-if="second.children && second.children.length > 0">
              <li v-for="(third, ti) in second.children"
                  :style="liStyle">
                <div @click.stop="callback([index,si,ti])">
                  <i :class="third.icon"></i>
                  <span>{{third.text}}</span>
                </div>
              </li>
            </ul>
        </li>
        </ul>
      </li>
    </ul>
  </div>
</template>

<script>
  export default {
    name: 'EasyCm',
    data() {
      return {
        // 是否显示
        show: false,
        // 触发点坐标
        axis: {
          x: 0,
          y: 0
        }
      }
    },
    props: {
      tag: {},
      list: {
        required: true
      },
      // 是否开启下划线
      underline: {
        default: false
      },
      // 是否开启箭头
      arrow: {
        default: false
      },
      // 是否开启边界检测
      border: {
        default: true
      },
      // 列表项宽度
      itemWidth: {
        default: 140
      },
      // 列表项高度
      itemHeight: {
        default: 36
      },
      // 列表项字体
      itemSize: {
        default: 14
      },
      // 显示点偏移量
      offset: {
        default: () => {
          return {
            x: 6, y: 2
          }
        }
      },
      // 边界距离
      borderWidth: {
        default: 6
      }
    },
    mounted() {
      this.$root.$on('easyAxis', (axis) => {
        if (axis.tag == this.tag){
            this.show = true
            this.axis = axis
            this.$nextTick(() => {
            const firstMenuItem = this.$el.querySelector('.cm-ul li:first-child div');
            if (firstMenuItem) {
              firstMenuItem.focus();
            }});
        }
      })
      document.addEventListener('click', () => {
          this.show = false
      }, true)
      
    },
    watch: {
      axis() {
        if (this.border){
          let bw = document.body.offsetWidth, bh = document.body.offsetHeight
          if (this.axis.x + this.offset.x + this.itemWidth >= bw) {
            this.axis.x = bw - this.itemWidth - this.borderWidth - this.offset.x
          }
          if (this.axis.y + this.offset.y + this.itemHeight * this.list.length >= bh) {
            this.axis.y = bh - this.itemHeight * this.list.length - this.borderWidth -this.offset.y
          }
        }
      }
    },
    computed: {
      axisComputed() {
        return {
          top: this.axis.y + this.offset.y + 'px',
          left: this.axis.x + this.offset.x + 'px'
        }
      },
      liStyle() {
        return{
          width: this.itemWidth + 'px',
          height: this.itemHeight + 'px',
          lineHeight: this.itemHeight + 'px',
          fontSize: this.itemSize + 'px'
        }
      },
      firstLeft() {
        let bw = document.body.offsetWidth
        return this.axis.x + this.itemWidth*2 >= bw
      },
      secondLeft() {
        let bw = document.body.offsetWidth
        return this.axis.x + this.itemWidth*3 >= bw
      }
    },
    methods: {
      secondBorderCheck(i){
        let bw = document.body.offsetWidth, bh = document.body.offsetHeight
        let cy = this.axis.y + ( i + this.list[i].children.length )* this.itemHeight
        return {
          left: this.axis.x + this.itemWidth*2 >= bw ? '-100%' : '100%',
          top: bh >= cy ? 0 : -(this.list[i].children.length - 1)* this.itemHeight + 'px'
        }
      },
      thirdBorderCheck(i,si){
        let bw = document.body.offsetWidth, bh = document.body.offsetHeight
        let cy =  this.axis.y + this.list[i].children[si].children.length * this.itemHeight + (i+si)*this.itemHeight+parseInt(this.secondBorderCheck(i).top)
        return {
          left: this.axis.x + this.itemWidth*3 >= bw ? '-100%' : '100%',
          top: cy > bh ? -(this.list[i].children[si].children.length - 1) * this.itemHeight + 'px' : 0
        }
      },
      callback (indexList){
        this.$emit('ecmcb',indexList)
      },
      handleMouseover(event){
        if (this.show) {
          const elementToFocus = event.target;
          elementToFocus.focus();
        }
      },
      handleKeyDown(event) {
        if (!this.show) return;

        event.preventDefault();
        //set up all the common default variables
        const container = this.$refs.contextMenuContainer;
        const focusedElement = document.activeElement;
        const submenu = focusedElement.nextElementSibling;

        //returns true if element is a submenu
        const checkSubmenu=(element)=>{
          return element && element.classList.contains('cm-ul-2');
        }

        const handleSubMenuNavigation = () => {
          const submenuItems = container.querySelectorAll('.cm-ul-2 > li > div');
          const subIndex = Array.from(submenuItems).findIndex(item => item === focusedElement);
          const subNextIndex = subIndex >= 0 ? (event.key === 'ArrowDown' ? (subIndex < submenuItems.length - 1 ? subIndex + 1 : 0) : (subIndex > 0 ? subIndex - 1 : submenuItems.length - 1)) : 0;
          submenuItems[subNextIndex].focus();
        };

        const handleMainMenuNavigation = () => {
          const menuItems = Array.from(container.children[1].children).map(li => li.querySelector('div:first-child'));
          const index = Array.from(menuItems).findIndex(item => item === focusedElement);
          const nextIndex = index >= 0 ? (event.key === 'ArrowDown' ? (index < menuItems.length - 1 ? index + 1 : 0) : (index > 0 ? index - 1 : menuItems.length - 1)) : 0;
          if (checkSubmenu(submenu)) {
            submenu.style.display = 'none';
          }
          menuItems[nextIndex].focus();
        };

        const selectElement = (elementToSelect)=>{
          elementToSelect.focus();
          if(elementToSelect.nextElementSibling){ //for cases when the menu item has submenu
            elementToSelect.nextElementSibling.style.display = 'block';
          }else{ //for cases when item has no submenu
            setTimeout(() => {
              elementToSelect.click();
            }, 70);
          }
        }

        const handleShortcut = (key)=>{
          let elementsWithShortcuts = container.querySelectorAll('.shortcut');
          //for cases when this is submenu
          if (checkSubmenu(submenu) && !['none', ''].includes(submenu.style.display)) {
            elementsWithShortcuts = submenu.querySelectorAll('.sub-shortcut');
          }
          const foundElement = Array.from(elementsWithShortcuts).find(item=>item.textContent === key);
          if(foundElement){
            const elementToFocus = foundElement.parentElement;
            selectElement(elementToFocus);
          }
        };


        switch (event.key) {
          case 'ArrowDown':
          case 'ArrowUp':
            if (focusedElement && checkSubmenu(focusedElement.parentElement.parentElement)) {
              handleSubMenuNavigation();
            } else {
              handleMainMenuNavigation();
            }
            break;

          case 'Enter':
            if (checkSubmenu(submenu)) {
              submenu.style.display = 'block';
            } else {
              focusedElement.click();
            }
            break;

          case 'ArrowRight':
          case 'ArrowLeft':
            if (checkSubmenu(submenu)) {
              submenu.style.display = 'block';
              const firstSubmenuItem = submenu.querySelector('.cm-ul-2 li:first-child div');
              if (firstSubmenuItem) {
                firstSubmenuItem.focus();
              }
            } else {
              const parentElement = focusedElement.parentElement.parentElement;
              if (checkSubmenu(parentElement)) {
                const mainMenuActiveItem = parentElement.previousElementSibling;
                parentElement.style.display = 'none';
                mainMenuActiveItem.focus();
              }
            }
            break;
          default:
            //handle other keys
            let key;
            if(event.key >="0" && event.key <="9"){
              key = event.key;
            }else if(event.code >= "KeyA" && event.code <= "KeyZ"){
              // use event.code to prevent the keyboard from having another language letter
              key = event.code.replace("Key",""); 
            }
            if(key !== undefined){
              handleShortcut(key);
            }
            break;
        }
      }
    }
  }
</script>

<style scoped>
  .icon {
    width: 0.9em; height: 0.9em;
    vertical-align: -0.15em;
    fill: currentColor;
    overflow: hidden;
    position: absolute;
    right: 0%;
    top: 50%;
    transform: translateY(-50%);
  }
  .cm-container {
    position: fixed;
    user-select: none;
    -ms-user-select: none;
    -webkit-user-select: none;
    -moz-user-select: none;
    z-index: 9999;
  }
  .cm-ul {
    width: 100%;
    padding: 0;
    margin: 0;
    list-style: none;
    box-shadow: 0 0 1px #666;
    background-color: #ffffff;
    color: #2e2e2e;
  }

  .cm-ul li {
    width: 100%;
    box-sizing: border-box;
    text-align: left;
    position: relative;
    cursor: pointer;
  }
  .cm-ul li:hover>ul{
    display: block;
  }
  .cm-ul li div{
    display: inline-block;
    box-sizing: border-box;
    width: 100%;
    height: 100%;
    padding: 0 0.8em;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    position: relative;
  }
  .cm-ul li i {
    display: inline-block;
    width: 1em;
    font-size: 1.3em;
    text-align: center;
  }
  .cm-ul li div:hover{
    background-color: #989898;
    outline-width: 0;
    color: #fff;
  }
  .cm-ul li div:focus-visible{
    background-color: #666666;
    outline-width: 0;
    color: #fff;
  }
  .cm-ul-2,.cm-ul-3 {
    position: absolute;
    top: 0;
    display: none;
    z-index: 10000;
  }
  .cm-left svg{
    transform: translateY(-50%) rotate(180deg) ;
    left: 0;
  }
  .cm-underline li div:after{
    content: '';
    width: 90%;
    position: absolute;
    left: 5%;
    top: 0;
    height: 1px;
    background-color: #cccccc;
    z-index: 10001;
  }
  .cm-underline li div:hover:after,.cm-underline>li:first-child>div:after{
    display: none;
  }
</style>

<template>
  <v-easy-dialog v-model="viewOrder" fullscreen backdropClass="order" focus-on="#input-name">
    <div class= overflow-y-auto">
      <div class="order__container">
        <transition-group class="order-inner" name="fade" duration="600" tag="div">
          <div v-if="successForm" class="order__success" key="success">
            <h3 class="order__success__title">Ваша заявка принята</h3>
            <p class="order__success__text">Наш менеджер свяжется с&nbsp;вами в&nbsp;ближайшее время</p>
            <button type="button" class="order-form__submit order__success__submit" @click="hidePopUp">Ясно</button>
          </div>
          <template v-else-if="viewOrderForm">
            <h2 class="order-title" key="order-title">Заказ услуг</h2>
            <div class="order-slugs" key="order-slugs">
              <h4 class="order-slugs__title">Проект</h4>
              <ul class="order-slugs__list">
                <li v-for="slug in slugs" :key="slug.id" :class="[
                    'order-slugs__list__item',
                    { 'order-slugs__list__item_active': slug.active }
                ]">
                  <button type="button" v-html="slug.title" @click.stop="submitSlug(slug)" />
                </li>
              </ul>
            </div>
            <div class="order-form" key="order-form">
              <div class="order-form__group">
                <input id="input-name" placeholder="Имя" class="input order-form__group__input" name="company" />
              </div>
              <div class="order-form__group">
                <input placeholder="Телефон или удобный мененджер" class="input order-form__group__input" name="phone" />
              </div>
              <div class="order-form__group">
                <input placeholder="Электропочта" class="input order-form__group__input" name="mail" />
              </div>
              <button type="button" class="order-form__submit" @click="submitForm">Отправить заявку</button>
            </div>
          </template>
        </transition-group>

        <transition-group name="fade" duration="600">
          <button v-if="viewOrder && viewOrderContent" type="button" class="order-close" @click="hidePopUp" key="order-close">
            <img class="order-close__icon" src="../assets/images/order_close.svg" alt="Close" />
          </button>
          <iframe v-if="viewOrder && viewOrderContent" key="popupVideo" id="popupVideo" ref="popupVideo" class="order__video" src="https://player.vimeo.com/video/820383465?autoplay=0&loop=1&autopause=0&muted=0&controls=0" width="640" height="360" frameborder="0" allow="autoplay; fullscreen" allowfullscreen />
        </transition-group>
      </div>
    </div>
  </v-easy-dialog>
</template>

<script>
import { mapMutations, mapGetters } from 'vuex';

export default {
  name: 'TheOrder',
  data: () => ({
    slugs: [
      { id: 1, title: 'Имиджевый видеоролик', active: true },
      { id: 2, title: 'Видеоблог', active: false },
      { id: 3, title: 'Вебинар', active: true },
      { id: 4, title: 'Подкаст', active: false },
    ],
    viewOrder: false,
    viewOrderContent: false,
    player: null,
    successForm: false,
  }),
  computed: {
    ...mapGetters({
      orderPopup: 'page/orderPopup',
    }),
    viewOrderForm() {
      return Boolean(this.viewOrder && this.viewOrderContent && !this.successForm);
    },
  },
  watch: {
    orderPopup: {
      async handler(value) {
        if (value) {
          this.viewOrder = true;
          await this.orderTick(300);
          this.viewOrderContent = true;
          // this.viewOrder = true;
          await this.orderTick(300);
          if (!this.player) await this.initializeVideo();
          await this.playVideo();

        } else {
          this.successForm = false;
          this.viewOrder = false;
          this.viewOrderContent = false;

          if (this.player) {
            this.player.destroy();
            this.player = null;
          }
        }
      }, deep: true, immediate: true,
    }
  },
  mounted() {
  },
  methods: {
    ...mapMutations({
      setOrderPopup: 'page/SET_ORDER_POPUP',
    }),
    orderTick(ms = 100) {
      return new Promise(resolve => this.$nextTick(() => setTimeout(() => resolve(), ms)));
    },
    initializeVideo() {
      return new Promise((resolve) => {
        this.player = new Vimeo.Player(this.$refs.popupVideo);
        this.player.ready().then(() => resolve());
      });
    },
    playVideo() {
      return new Promise((resolve) => {
        this.player.play().then(() => resolve()).catch(e => console.log(e));
      });
    },
    submitSlug(slug) {
      slug.active = Boolean(!slug.active);
    },
    submitForm() {
      this.successForm = true;
    },
    async hidePopUp() {
      this.viewOrderContent = false;
      this.setOrderPopup(false);
    },
  },
}
</script>

<style lang="scss">

.fade {
  &-enter {
    opacity: 0;

    &-to {
      opacity: 1;
    }
    &-active {
      transition: all var(--transition-time) 0s var(--transition-timing)!important;
    }
  }
  &-leave {
    opacity: 1;

    &-active {
      transition: all var(--transition-time) 0s var(--transition-timing)!important;
    }
    &-to {
      opacity: 0;
    }
  }
}
</style>
<template>
	<div class="side-menu" :class="[menuDirectionClass]" :style="menuStyles">
		<div class="menu-wrapper">
			<slot></slot>
		</div>
		<div class="toggler" @click="toggle">
			<icon name="chevron-left"></icon>
		</div>
	</div>
</template>

<script>
  import TWEEN from '@tweenjs/tween.js';
  import 'vue-awesome/icons/chevron-left';
  import Icon from 'vue-awesome/components/Icon.vue';
  import _ from 'lodash';

  export default {
    name: 'side-menu',
    components: { Icon },
    data() {
      return {
        opened: true,
        positionOffset: 0,
        originalStyles: {},
      };
    },
    props: {
      side: {
        type: String,
        default: 'left',
      },
      duration: {
        type: Number,
        default: 500,
      },
      remember: {
        type: Boolean,
        default: true,
      },
      autoClose: {
        type: Boolean,
        default: false,
      },
      pushBody: {
        type: Boolean,
        deafult: true,
      },
      closeOnClick: {
        type: Boolean,
        default: true,
      },
      width: {
        type: String,
        default: '15%',
      },
      sliderStyles: {
        type: Object,
        default: () => {
        },
      },
    },
    computed: {
      menuDirectionClass() {
        return `side-menu-${this.side}`;
      },
      menuStyles() {
        const styles = {
          width: this.width,
        };
        styles[this.side] = `${this.positionOffset}px`;

        return _.assign(styles, this.sliderStyles);
      },
    },
    methods: {
      toggle() {
        if (this.opened) {
          this.closeMenu();
        } else {
          this.openMenu();
        }
      },
      closeMenu() {
        if (this.side === 'left') {
//          if (this.pushBody) {
//
//          }

          this.opened = false;

          this.tweenPositionOffset();
        }
      },
      openMenu() {
        this.opened = true;

        this.tweenPositionOffset();
      },
      tweenPositionOffset() {
        const vm = this;
        const from = {
          positionOffset: this.positionOffset,
          bodyOffset: this.removePixelUnitTrailing(document.body.style.marginLeft),
        };
        const to = {
          positionOffset: 0,
          bodyOffset: 0,
        };

        if (this.opened) { // opening
          to.positionOffset = 0;
          to.bodyOffset = this.getOriginalStylesPixelVal('width') + 20;
        } else { // closing
          to.positionOffset = -(this.getOriginalStylesPixelVal('width') + 2);
          to.bodyOffset = this.getOriginalStylesPixelVal('marginLeft');
        }

        function animate() {
          if (TWEEN.update()) {
            requestAnimationFrame(animate);
          }
        }

        new TWEEN.Tween(from)
          .easing(TWEEN.Easing.Quadratic.Out)
          .to(to, this.duration)
          .onUpdate(function update() {
            vm.positionOffset = this.positionOffset;
            document.body.style.marginLeft = `${this.bodyOffset}px`;
          })
          .start();
        animate();
      },
      getOriginalStylesPixelVal(prop) {
        const value = this.originalStyles.getPropertyValue(prop);

        return this.removePixelUnitTrailing(value);
      },
      /* eslint-disable no-param-reassign */
      removePixelUnitTrailing(value) {
        if (value.endsWith('px')) {
          value = value.substring(0, value.length - 2);
        }

        return Number(value);
      },
    },
    mounted() {
      this.originalStyles = window.getComputedStyle(this.$el);

      if (!this.opened) {
        this.positionOffset = -(this.getOriginalStylesPixelVal('width') + 2);
      }
    },
  };
</script>

<style scoped lang="scss">
	@mixin background-clip {
		-webkit-background-clip: padding-box;
		background-clip: padding-box;
	}

	@mixin menu-direction-styles($side) {
		$open-direction: if($side == 'left', 'right', 'left');
		$close-direction: $side;
		$shadow-multiplier: if($side == 'left', 1, -1);

		&.side-menu-#{$close-direction} {
			#{$close-direction}: 0;
			#{$open-direction}: auto;

			> .toggler {
				-webkit-border-top-#{$open-direction}-radius: 4px;
				-webkit-border-bottom-#{$open-direction}-radius: 4px;
				-moz-border-radius-top#{$open-direction}: 4px;
				-moz-border-radius-bottom#{$open-direction}: 4px;
				border-top-#{$open-direction}-radius: 4px;
				border-bottom-#{$open-direction}-radius: 4px;
				border-#{$close-direction}: 1px solid #fff;
				-webkit-box-shadow: 1*$shadow-multiplier px 0px 8px rgba(0, 0, 0, .175);
				box-shadow: 1*$shadow-multiplier px 0px 8px rgba(0, 0, 0, .175);
				#{$open-direction}: -20px;
			}
		}
	}

	@mixin solid-border {
		border: 1px solid rgba(0, 0, 0, .15);
	}

	$background-color: #fff;

	.side-menu {
		@include background-clip;
		@include solid-border;

		z-index: 999999;
		position: fixed;
		top: -1px;
		bottom: -1px;
		padding: 0;
		background-color: $background-color;
		-webkit-box-shadow: 0 6px 12px rgba(0, 0, 0, .175);
		box-shadow: 0 6px 12px rgba(0, 0, 0, .175);
		min-width: unset;

		> .menu-wrapper {
			padding: 0;
			position: absolute;
			bottom: 0;
			top: 0;
			overflow: auto;
			width: 100%;
		}

		.toggler {
			@include background-clip;
			@include solid-border;

			width: 20px;
			height: 48px;
			position: absolute;
			top: 47%;
			cursor: pointer;
			background-color: $background-color;

			> span {
				margin: 15px 2px;
			}
		}

		@include menu-direction-styles('left');
		@include menu-direction-styles('right');
	}
</style>

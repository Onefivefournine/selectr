<style lang="scss">
$gray:#636c72;
$dgray: #292b2c;
$lgray: #eceeef;
$primary:#6ae2a2;
$anti-primary:#ff482a;
.selectr {
  &__button.form-control {
    height: auto;
    &:focus {
      outline: none;
      box-shadow: none;
      -webkit-box-shadow: none;
      border: 1px solid $lgray;
    }
  }
  &__placeholder {
    color: $gray;
    float: left;
  }
  &__arrow {
    fill: $dgray;
    width: 8px;
    right: 18px;
    top: 48%;
    position: absolute;
  }
  &__match-container {
    float: left;
    max-width: 100%;
  }
  &__match {
    float: left;
    padding: 1px 6px;
    background: $lgray;
    color: $gray;
    border: 1px solid lighten($gray, 20%);
    border-radius: 4px;
    margin: -2px 2px -2px 0;
    &:first-child {
      margin-left: -8px;
    }
  }
  &__options {
    position: absolute;
    z-index: 1;
    background: #fff;
    border: 1px solid $lgray;
    border-bottom-left-radius: 4px;
    border-bottom-right-radius: 4px;
    margin-top: -1px;
    border-top: 1px solid #fff;
    width: 100%;
    top: calc(100% - 1px);
    left: 0;
    text-align: left;
    overflow: auto;
    max-height: 140px;
  }
  &__option {
    padding: 5px 10px;
    cursor: pointer;
    &:hover {
      background-color: $primary;
      color: #fff;
    }
    &.disabled {
      background-color: $lgray;
      color: $gray;
      &:hover {
        background-color: $anti-primary;
        color: #fff;
      }
    }
  }
  &__container {
    position: relative;
  }
  &.opened &__button {
    border-bottom-left-radius: 0;
    border-bottom-right-radius: 0;
  }
  &.opened &__arrow {
    display: none;
  }
}
</style>
<template>
  <div class="selectr__container selectr"
    :class="opened?'opened':''">
    <button :class="className + ' selectr__button'"
      @blur="blur()"
      @click.prevent="toggle($event)">
      <div v-if="selectedItems.length"
        class="selectr__match-container">
        <span v-for="match in selectedItems"
          class="selectr__match"
          @click="deselect(match)">{{match.label}}</span></div>
      <span v-else
        class="selectr__placeholder">{{placeholder}}</span>
      <svg viewBox="0 0 10 10"
        xmlns="http://www.w3.org/2000/svg"
        class="selectr__arrow">
        <path d="M0,0 L10,0 L5,5 L0,0Z"></path>
      </svg>
      <div class="selectr__options"
        v-show="opened">
        <div class="selectr__option"
          :class="option.selected?'disabled':''"
          @click="select(option)"
          :key="option.label"
          v-for="option in innerOptions">
          {{option.label}}
        </div>
      </div>
    </button>
  </div>
</template>
<script>
import Vue from 'vue';

export default Vue.extend( {
  data() {
    return {
      selectedItems: [],
      innerOptions: [],
      opened: false,
    }
  },
  props: {
    className: {
      type: String,
      default: 'form-control'
    },
    placeholder: {
      type: String,
      default: ''
    },
    options: {
      type: Array,
      default () {
        return [];
      },
      validator( value ) {
        return value.filter( ( el ) => {
          return typeof el === 'string' || Object.prototype.toString.call( el ) === '[object Object]'
        } ).length;
      }
    },
    label: {
      type: String,
      default: 'label'
    },
    value: {
      default: []
    },
    oneAtTime: {
      type: Boolean,
      default: true
    },
    getOptionLabel: {
      type: Function,
      default ( option, mutatedLabel ) {
        let currentLabel = mutatedLabel || this.label;
        if ( typeof option === 'object' ) {
          if ( currentLabel && typeof currentLabel === 'string' && option[ currentLabel ] ) {
            return option[ currentLabel ]
          } else if ( currentLabel.includes( '.' ) ) {
            mutatedLabel = currentLabel.split( '.' )
            let mutatedOption = option[ mutatedLabel[ 0 ] ];
            mutatedLabel.shift();
            mutatedLabel = mutatedLabel.join( '.' );
            return this.getOptionLabel( mutatedOption, mutatedLabel )
          }
        }
        return option;
      }
    },
  },
  watch: {
    selectedItems( nw, old ) {
      this.$emit( 'input', nw.map( ( el ) => el.value ) )
    }
  },
  created() {
    this.innerOptions = this.options.map( ( opt ) => {
      let retVal = {};
      if ( typeof opt === 'string' ) {
        retVal = {
          label: opt,
          value: opt,
          selected: false,
          innerId: ( Math.random() * 100000 ).toFixed() // TODO replace by uuid
        }
      } else if ( typeof opt === 'object' ) { // dirty check, because of props validation
        retVal = {
          label: this.getOptionLabel( opt ),
          value: opt,
          selected: false,
          innerId: ( Math.random() * 100000 ).toFixed()
        }
      }
      return retVal;
    } )
  },
  methods: {
    blur() {
      if ( this.opened ) this.close()
    },
    open() {
      this.opened = true;
    },
    close() {
      this.opened = false;
    },
    toggle( ev ) {
      if ( !ev.target.classList.contains( 'selectr__match' ) ) this.opened = !this.opened;
    },
    deselect( option ) {
      let i = this.selectedItems.map( ( el ) => el.innerId ).indexOf( option.innerId );
      if ( ~i ) {
        this.selectedItems.splice( i, 1 );
        option.selected = false;
      }
    },
    select( option ) {
      if ( !this.oneAtTime ) {
        this.selectedItems.push( option );
      } else if ( !option.selected ) {
        let i = this.selectedItems.map( ( el ) => el.innerId ).indexOf( option.innerId );
        if ( !~i ) {
          this.selectedItems.push( option );
          option.selected = true;
        }
      } else if ( option.selected ) {
        this.deselect( option )
      }
    },
  },
  template
} );
</script>

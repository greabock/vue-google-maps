<template>
    <label>
        <span v-text="label"></span>
        <input type="text" :placeholder="placeholder" :class="className"
          ref="input"/>
    </label>
</template>

<script>
  import _ from 'lodash'
  import eventBinder from '../utils/eventsBinder.js'
  import propsBinder from '../utils/propsBinder.js'
  import downArrowSimulator from '../utils/simulateArrowDown.js'
  import getPropsValuesMixin from '../utils/getPropsValuesMixin.js'
  import {loaded} from '../manager.js'
  import assert from 'assert';

  const props = {
      bounds: {
        type: Object,
      },
      defaultPlace: {
        type: String,
        default: '',
      },
      componentRestrictions: {
        type: Object,
        default: null,
      },
      types: {
        type: Array,
        default: function() { return []; }
      },
      placeholder: {
        required: false,
        type: String
      },
      className: {
          required: false,
          type: String
      },
      label: {
          required: false,
          type: String,
          default: null
      },
      selectFirstOnEnter: {
        require: false,
        type: Boolean,
        default: false
      }
  }

  export default {
    mixins: [getPropsValuesMixin],

    mounted() {
      const input = this.$refs.input;
      input.value = this.defaultPlace;
      loaded.then(() => {
        window.i = input;
        const options = _.clone(this.getPropsValues());
        if (this.selectFirstOnEnter) {
          downArrowSimulator(this.$refs.input);
        }

        assert(typeof(google.maps.places.Autocomplete) === 'function',
          "google.maps.places.Autocomplete is undefined. Did you add 'places' to libraries when loading Google Maps?")

        this.autoCompleter = new google.maps.places.Autocomplete(this.$refs.input, options);
        propsBinder(this, this.autoCompleter, _.omit(props, ['placeholder', 'place', 'selectFirstOnEnter']));

        this.autoCompleter.addListener('place_changed', () => {
          this.$emit('place_changed', this.autoCompleter.getPlace())
        })

      })
    },
    props: props,
  }
</script>

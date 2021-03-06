<template>
    <div class="border-b border-40">
        <div class="flex">
            <div class="w-1/5 py-6 px-8">
                <label class="inline-block text-80 pt-2 leading-tight">
                    {{field.name}}
                </label>
            </div>
            <div class="w-1/2 py-6 px-8">
                <el-alert
                        v-if="attributes.length === 0"
                        title="该分类暂无匹配属性"
                        type="error"
                        :closable="false">
                </el-alert>
            </div>
        </div>
        <translation-form-field
                :errors="errors"
                v-for="attribute in attributes" :key="attribute.name" :field="attribute">
        </translation-form-field>
    </div>

</template>

<script>
  import { FormField, HandlesValidationErrors, Minimum } from 'laravel-nova'
  import TranslationFormField from './TranslationFormField'
  import Helper from '../helper'

  export default {
    mixins: [FormField, HandlesValidationErrors, Helper],

    props: ['resourceName', 'resourceId', 'field'],

    components: {
      TranslationFormField
    },

    data () {
      return {
        eventResponse: null,
        params: 'taxon',
      }
    },

    methods: {
      /*
       * Set the initial, internal value for the field.
       */
      setInitialValue () {
        this.value = this.field.value || ''
      },

      /**
       * Fill the given FormData object with the field's internal value.
       */
      fill (formData) {
        this.attributes.forEach(attribute => {
          attribute.fill(formData)
        })
        // formData.append(this.field.attribute, this.value || '')
      },

      /**
       * Update the field's internal value.
       */
      handleChange (value) {
        this.value = value
      },

      handleEventResponseChange () {
        this.fetchRequest().then(({data}) => {
          this.fillAttributesValues(data)
        })
      },

      registerListener () {
        Nova.$on(this.eventName, value => {
          if (value !== this.eventResponse) {
            this.eventResponse = value
            this.handleEventResponseChange()
          }
        })
      },

      deleteListener () {
        Nova.$off(this.eventName)
      },

      fetchRequest () {
        return Minimum(Nova.request().get(this.api, {
          params: {
            [this.params]: this.eventResponse
          }
        }))
      }
    },
    computed: {
      api () {
        return `${Nova.config['erp-prefix']}/${_.get(this, 'field.api', 'product-attributes')}`
      },
      eventName () {
        return _.get(this, 'field.eventKey', 'taxon-change')
      }
    },
    mounted () {
      this.registerListener()
      this.params = _.get(this, 'field.params', this.params)
    },
    beforeDestroy () {
      this.deleteListener()
    }
  }
</script>

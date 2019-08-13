<template>
    <default-field :field="field" :full-width-content="true">
        <template slot="field">
            <nav  class="tabs">
                <ul>
                    <li
                            class="inline-block font-bold cursor-pointer animate-text-color select-none"
                            :class="{ 'text-60': localeKey !== currentLocale, 'text-primary': localeKey === currentLocale }"
                            :key="`a-${localeKey}`"
                            v-for="(locale, localeKey) in field.locales"
                            @click="changeTab(localeKey)"
                    >
                        {{ locale }}
                    </li>
                </ul>
            </nav>
            <div v-if="!field.singleLine && !field.trix && !field.tinymce" class="mt-4 p-2">
                <textarea
                        ref="field"
                        :id="field.name"
                        class=" w-full form-control form-input form-input-bordered py-3 min-h-textarea"
                        :class="errorClasses"
                        :placeholder="field.name"
                        v-model="value[currentLocale]"
                        @keydown.tab="handleTab"
                ></textarea>
            </div>

            <div v-if="!field.singleField && field.trix" class="mt-4  p-2">
                <trix
                    ref="field"
                    name="trixman"
                    :value="value[currentLocale]"
                    placeholder=""
                    @change="handleChange"
                />
            </div>

            <div v-if="!field.singleField && field.tinymce" class="mt-4 full p-2">
                <tiny-mce
                        ref="field"
                        name="tinyman"
                        :current_locale="currentLocale"
                        :field_value="value[currentLocale]"
                        :field="field"
                        placeholder="Placeholder"
                        @change="handleChange"
                />
            </div>
            <div v-if="field.singleLine" class="mt-4 p-2">
                <input
                    ref="field"
                    type="text"
                    :id="field.name"
                    class="w-full form-control form-input form-input-bordered"
                    :class="errorClasses"
                    :placeholder="field.name"
                    v-model="value[currentLocale]"
                    @keydown.tab="handleTab"
                />
            </div>

            <p v-if="hasError" class="my-2 text-danger">
                {{ firstError }}
            </p>
        </template>
    </default-field>
</template>

<script>

import Trix from '../Trix'
import TinymceVue from '@tinymce/tinymce-vue'
import TinyMce from '../TinyMce'

import { FormField, HandlesValidationErrors } from 'laravel-nova'
import { EventBus } from '../event-bus';

export default {
    mixins: [FormField, HandlesValidationErrors],

    props: ['resourceName', 'resourceId', 'field'],

    components: {TinymceVue, Trix, TinyMce },

    mounted() {
        this.currentLocale = this.locales[0] || null;
        EventBus.$on('localeChanged', locale => {
            if(this.currentLocale !== locale) {
                this.changeTab(locale, true);
            }
        });
    },

    data() {
        return {
            locales: Object.keys(this.field.locales),
            currentLocale: null
        }
    },

    methods: {
        /*
         * Set the initial, internal value for the field.
         */
        setInitialValue() {
          this.value = this.field.value || {};
          this.currentLocale = this.locales[0] || null;
        },

        /**
         * Fill the given FormData object with the field's internal value.
         */
        fill(formData) {
            Object.keys(this.value).forEach(locale => {
                formData.append(this.field.attribute + '[' + locale + ']', this.value[locale] || '')
            })
        },

        /**
         * Update the field's internal value.
         */
        handleChange(value) {
          this.value[this.currentLocale] = value
        },

        changeTab(locale, dontEmit) {
            if(this.currentLocale !== locale){
                if(!dontEmit){
                    EventBus.$emit('localeChanged', locale);
                }

                this.currentLocale = locale;

                this.$nextTick(() => {
                    if (this.field.trix) {
                        this.$refs.field.update();
                    }else if(this.field.tinymce){
                        this.$refs.field.update();
                    } else {
                        //this.$refs.field.focus();
                    }
                })
            }
        },

        handleTab(e) {
            const currentIndex = this.locales.indexOf(this.currentLocale)
            if (!e.shiftKey) {
                if (currentIndex < this.locales.length - 1) {
                    e.preventDefault();
                    this.changeTab(this.locales[currentIndex + 1]);
                }
            } else {
                if (currentIndex > 0) {
                    e.preventDefault();
                    this.changeTab(this.locales[currentIndex - 1]);
                }
            }
        }
    }
}
</script>

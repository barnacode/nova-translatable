<template>
    <editor ref="theEditor"
            v-model="value"
            :api-key="tinymce_api_key"
            :id="field.attribute"
            :class="errorClasses"
            :placeholder="placeholder"
            :init="options"
            @onKeyUp="onChange"
            @onInit="onInitialize"
    ></editor>
</template>

<script>
    import { FormField, HandlesValidationErrors } from 'laravel-nova'
    import Editor from '@tinymce/tinymce-vue'

    export default {
        name: 'tinymce-vue',
        components: { Editor },

        mounted:function(){

            // Default values
            this.locale = 'en_US';
            this.value = this.field.value[this.locale] || '';
            let field = this.field;

            tinyMCE.PluginManager.add('maxchars', function(editor) {

                //set a default value for the maxChars peroperty
                editor.maxChars = field.options.maxChars || 50000000;

                var label = null;

                function init() {
                    //add a custom style which will be injected into the iframe
                    editor.contentStyles.push(".maxchars {color: red !important;}");

                    //try and add label to to the status bar
                    var statusbar = editor.theme.panel && editor.theme.panel.find('#statusbar')[0];

                    if (statusbar) {
                        statusbar.insert({
                            type: 'label',
                            name: 'maxcharslabel',
                            style: "color:red;",
                            classes: 'wordcount' //this puts it on the right
                        }, 0);

                        //cache the newly created element
                        label = statusbar.find('#maxcharslabel')
                    }

                    //console.log("Max chars inititilaized: limit = %d", editor.maxChars);
                    updateStyle(); //check if the initial content is valid
                };

                function updateMsg(show) {
                    if (!label) {
                        return;
                    }
                    var msg = show ? editor.maxChars + " chars max" : "";
                    //console.log(panel);
                    label.text(msg);
                }

                function updateStyle() {
                    var content = editor.getContent({
                        format: 'html'
                    });
                    var $body = editor.$('.mce-content-body');

                    //add class to content body based on content length
                    if (content.length > editor.maxChars) {
                        $body.addClass("maxchars");
                        updateMsg(true);

                    } else {
                        $body.removeClass("maxchars");
                        updateMsg(false);
                    }

                };
                editor.on('init', init);
                editor.on("change keyUp", updateStyle);
            });

        },

        mixins: [FormField, HandlesValidationErrors],

        props: ['resourceName', 'resourceId', 'field', 'name', 'field_value', 'placeholder', 'current_locale'],

        data: function() {
            return {
                locale: this.current_locale || 'en_US',
                value : ''
            }
        },

        computed: {
            tinymce_api_key() {
                return Nova.config.tinymce_api_key
            },

            options() {
                let options = this.field.options

                if (options.use_lfm) {
                    options['file_browser_callback'] = this.filemanager
                }
                return options
            }
        },

        methods: {
            /*
             * Set the initial, internal value for the field.
             */
            setInitialValue() {
                this.value = this.field.value || ''
            },

            /**
             * Fill the given FormData object with the field's internal value.
             */
            fill(formData) {
                formData.append(this.field.attribute, this.value || '')
            },

            /**
             * Update the field's internal value.
             */
            handleChange(value) {
                this.value = value
            },

            filemanager(field_name, url, type, win) {
                let x = window.innerWidth || document.documentElement.clientWidth || document.getElementsByTagName('body')[0].clientWidth;
                let y = window.innerHeight|| document.documentElement.clientHeight|| document.getElementsByTagName('body')[0].clientHeight;

                let cmsURL = this.options.path_absolute + this.options.lfm_url + '?field_name=' + field_name;
                if (type == 'image') {
                    cmsURL = cmsURL + '&type=Images';
                } else {
                    cmsURL = cmsURL + '&type=Files';
                }

                tinyMCE.activeEditor.windowManager.open({
                    file : cmsURL,
                    title : 'Filemanager',
                    width : x * 0.8,
                    height : y * 0.8,
                    resizable : 'yes',
                    close_previous : 'no'
                });
            },
            onChange() {
                this.$emit('change', this.value);
            },
            update() {
                this.value = this.field.value[this.current_locale] || '';
                this.$refs.theEditor.value = this.value;
            },
            onInitialize() {
                this.value = this.field.value[this.locale] || '';
                this.$refs.theEditor.value = this.value;
            },

        }
    }
</script>

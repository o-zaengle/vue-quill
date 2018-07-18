<template>
    <div>
        <div class="ui attached segment" ref="quill" @click.prevent="focusEditor"></div>
    </div>
</template>

<script>
    import defaultsDeep from 'lodash.defaultsdeep'
    import Quill from 'quill'
    import GrammarlyInline from './formats/GrammarlyInline'
    import { ImageUpload } from 'quill-image-upload'

    export default {
        model: {
            prop: 'content',
        },

        props: {
            content: {},
            loadContent: false,
            formats: {
                type: Array,
                default() {
                    return []
                },
            },

            keyBindings: {
                type: Array,
                default() {
                    return []
                },
            },

            output: {
                default : 'html'
            },

            config: {
                default() {
                    return {}
                },
            },

        },

        data() {
            return {
                editor: {},
                defaultConfig: {
                    modules: {
                        toolbar: [
                            ['bold', 'italic', 'underline'],
                            [
                               { 'list': 'ordered' }, { 'list': 'bullet' }
                            ],
                            [{ 'align': [] }],
                            ['image'],
                        ],
                    },
                    theme: 'snow',
                },
            }
        },

        watch: {
            loadContent() {
                if(this.loadContent) {
                    this.setContent()
                }
            }
        },

        mounted() {
            if (this.keyBindings.length) {
                this.defaultConfig.modules.keyboard = {
                    bindings: this.keyBindings.map((binding) => {
                        if (binding.remove) return false
                        return {
                            key: binding.key,
                            metaKey: true,
                            handler: binding.method.bind(this),
                        }
                    })
                }
            }

            if (this.config.modules && this.config.modules.toolbar) {
                this.defaultConfig.modules.toolbar = []
            }

            Quill.register(GrammarlyInline)
            Quill.register('modules/imageUpload', ImageUpload);

            this.editor = new Quill(this.$refs.quill, defaultsDeep(this.config, this.defaultConfig))

            this.editor.on('text-change', (delta, source) => {
                this.$emit('text-change', this.editor, delta, source)
                this.$emit('input', this.output != 'delta' ? this.editor.root.innerHTML : this.editor.getContents())
            })

            this.editor.on('selection-change', (range, oldRange, source) => {
                this.$emit('selection-change', this.editor, {range, oldRange, source})
            })
        },

        methods: {
            setContent() {
                if(this.content.length) {
                    if (this.output === 'delta') {
                        this.editor.setContents(this.content)
                    } else {
                        this.editor.clipboard.dangerouslyPasteHTML(this.content)
                    }
                }
            },
            focusEditor(e) {
                if (!this.editor.hasFocus()) {
                    this.editor.focus()
                    this.editor.setSelection(this.editor.getLength()-1, this.editor.getLength())
                }
            }
        },
    }
</script>

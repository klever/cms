<template>
    <div :class="{'popover-open': isOpen}" v-on-clickaway="close" @mouseleave="leave">
        <div @click="toggle" ref="trigger" aria-haspopup="true" :aria-expanded="isOpen" v-if="$scopedSlots.default">
            <slot name="trigger"></slot>
        </div>
        <portal
            :to="portalTargetName"
            :target-class="targetClass"
        >
            <div
                ref="popover"
                :class="`popover-container ${isOpen ? 'popover-open' : ''}`"
                v-if="!disabled"
            >
                <div class="popover">
                    <div class="popover-content bg-white shadow-popover rounded-md">
                        <slot :close="close" :after-closed="afterClosed" />
                    </div>
                </div>
            </div>
        </portal>
    </div>
</template>

<script>
import { mixin as clickaway } from 'vue-clickaway';
import { createPopper } from '@popperjs/core';

export default {

    mixins: [ clickaway ],

    props: {
        disabled: {
            type: Boolean,
            default: false
        },
        placement: {
            type: String,
            default: 'bottom-end',
        },
        offset: {
            type: Array,
            default: () => [0, 10]
        },
        scroll: {
            type: Boolean,
            default: false
        },
        autoclose: {
            type: Boolean,
            default: false
        }
    },

    data() {
        return {
            isOpen: false,
            escBinding: null,
            popper: null,
            closedCallbacks: [],
            portalTarget: null,
        }
    },

    computed: {

        portalTargetName() {
            return this.portalTarget ? this.portalTarget.name : null;
        },

        targetClass() {
            return this.$vnode.data.staticClass;
        }

    },

    created() {
        this.createPortalTarget();
    },

    mounted() {
        if (! this.disabled) {
            this.$nextTick(() => this.bindPopper());
        }
    },

    beforeDestroy() {
        this.destroyPopper();
        this.destroyPortalTarget();
    },

    methods: {
        bindPopper() {
            this.popper = createPopper(this.$refs.trigger, this.$refs.popover, {
                placement: this.placement,
                modifiers: [
                    {
                        name: 'offset',
                        options: {
                            offset: this.offset
                        }
                    },
                    {
                        name: 'eventListeners',
                        options: {
                            scroll: this.scroll,
                            resize: true
                        }
                    }
                ]
            })
        },
        toggle() {
            this.isOpen ? this.close() : this.open();
        },
        open() {
            this.isOpen = true;
            this.escBinding = this.$keys.bind('esc', e => this.close())
            this.popper && this.popper.update();
        },
        close() {
            this.isOpen = false;
            if (this.escBinding) {
                this.escBinding.destroy();
            }
        },
        leave() {
            if (this.autoclose) {
                this.close();
            }
        },
        destroyPopper() {
            if (!this.popper) return;

            this.popper.destroy();
            this.popper = null;

            // run any after-closed callbacks
            this.closedCallbacks.forEach(callback => callback());
        },
        afterClosed(callback) {
            this.closedCallbacks.push(callback);
        },

        createPortalTarget() {
            let key = `popover-${this._uid}`;
            let portalTarget = { key, name: key };
            this.$root.portals.push(portalTarget);
            this.portalTarget = portalTarget;
        },

        destroyPortalTarget() {
            const i = _.findIndex(this.$root.portals, (portal) => portal.key === this.portalTarget.key);
            this.$root.portals.splice(i, 1);
        }
    }
}
</script>

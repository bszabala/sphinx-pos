<template>
    <div id="layout-config" :class="containerClass">
        <a href="#" class="layout-config-button" id="layout-config-button" @click="toggleConfigurator">
            <i class="pi pi-cog"></i>
        </a>
        <Button class="p-button-danger layout-config-close p-button-rounded p-button-text" icon="pi pi-times"
                @click="hideConfigurator"></Button>

        <div class="layout-config-content">
            <h6 class="mt-0">Component Scale</h6>
            <div class="config-scale">
                <Button icon="pi pi-minus" @click="decrementScale()" class="p-button-text"
                        :disabled="scale === scales[0]"/>
                <i class="pi pi-circle-on" v-for="s of scales" :class="{'scale-active': s === scale}" :key="s"/>
                <Button icon="pi pi-plus" @click="incrementScale()" class="p-button-text"
                        :disabled="scale === scales[scales.length - 1]"/>
            </div>

            <h6>Input Style</h6>
            <div class="p-formgroup-inline">
                <div class="field-radiobutton">
                    <RadioButton id="input_outlined" name="inputstyle" value="outlined"
                                 :modelValue="$primevue.config.inputStyle" @change="changeInputStyle('outlined')"/>
                    <label for="input_outlined">Outlined</label>
                </div>
                <div class="field-radiobutton">
                    <RadioButton id="input_filled" name="inputstyle" value="filled"
                                 :modelValue="$primevue.config.inputStyle" @change="changeInputStyle('filled')"/>
                    <label for="input_filled">Filled</label>
                </div>
            </div>

            <h6>Ripple Effect</h6>
            <InputSwitch :modelValue="rippleActive" @update:modelValue="changeRipple"/>

            <h6>Menu Type</h6>
            <div class="p-formgroup-inline">
                <div class="field-radiobutton">
                    <RadioButton id="static" name="layout" value="static" v-model="layout"
                                 @change="changeLayout($event, 'static')"/>
                    <label for="static">Static</label>
                </div>
                <div class="field-radiobutton">
                    <RadioButton id="overlay" name="layout" value="overlay" v-model="layout"
                                 @change="changeLayout($event, 'overlay')"/>
                    <label for="overlay">Overlay</label>
                </div>
            </div>

            <h6>Theme Mode</h6>
            <div class="p-formgroup-inline">
                <div class="field-radiobutton">
                    <RadioButton id="light" name="layoutThemeMode" value="light" v-model="themeMode"
                                 @change="changeThemeMode('light')"/>
                    <label for="light">Light</label>
                </div>
                <div class="field-radiobutton">
                    <RadioButton id="dark" name="layoutThemeMode" value="dark" v-model="themeMode"
                                 @change="changeThemeMode('dark')"/>
                    <label for="dark">Dark</label>
                </div>
            </div>

            <h6>Theme Color</h6>
            <div class="flex">
                <div style="width:2rem;height:2rem;border-radius:6px" class="bg-indigo-500 mr-3 cursor-pointer"
                     @click="changeThemeColor('indigo')"></div>
                <div style="width:2rem;height:2rem;border-radius:6px" class="bg-purple-500 mr-3 cursor-pointer"
                     @click="changeThemeColor('deeppurple')"></div>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props: {
        layoutMode: {
            type: String,
            default: 'static'
        },
        layoutThemeMode: {
            type: String,
            default: 'light'
        },
        layoutThemeColor: {
            type: String,
            default: 'indigo'
        }
    },
    data() {
        return {
            active: false,
            layout: this.layoutMode,
            scale: 13,
            scales: [11, 12, 13, 14, 15],
            themeMode: this.layoutThemeMode,
            themeColor: this.layoutThemeColor
        }
    },
    watch: {
        $route() {
            if (this.active) {
                this.active = false;
                this.unbindOutsideClickListener();
            }
        },
        layoutMode(newValue) {
            this.layout = newValue;
        },
        layoutThemeMode(newValue) {
            this.themeMode = newValue;
        }
    },
    outsideClickListener: null,
    methods: {
        toggleConfigurator(event) {
            this.active = !this.active;
            event.preventDefault();

            if (this.active)
                this.bindOutsideClickListener();
            else
                this.unbindOutsideClickListener();
        },
        hideConfigurator(event) {
            this.active = false;
            this.unbindOutsideClickListener();
            event.preventDefault();
        },
        changeInputStyle(value) {
            this.$primevue.config.inputStyle = value;
        },
        changeRipple(value) {
            this.$primevue.config.ripple = value;
        },
        changeLayout(event, layoutMode) {
            this.$emit('layout-change', layoutMode);
            event.preventDefault();
        },
        changeLayoutColor(event, layoutColor) {
            this.$emit('layout-color-change', layoutColor);
            event.preventDefault();
        },
        bindOutsideClickListener() {
            if (!this.outsideClickListener) {
                this.outsideClickListener = (event) => {
                    if (this.active && this.isOutsideClicked(event)) {
                        this.active = false;
                    }
                };
                document.addEventListener('click', this.outsideClickListener);
            }
        },
        unbindOutsideClickListener() {
            if (this.outsideClickListener) {
                document.removeEventListener('click', this.outsideClickListener);
                this.outsideClickListener = null;
            }
        },
        isOutsideClicked(event) {
            return !(this.$el.isSameNode(event.target) || this.$el.contains(event.target));
        },
        decrementScale() {
            this.scale--;
            this.applyScale();
        },
        incrementScale() {
            this.scale++;
            this.applyScale();
        },
        applyScale() {
            document.documentElement.style.fontSize = this.scale + 'px';
        },
        changeTheme() {
            this.$primevue.changeTheme(
                `mdc-${this.layoutThemeMode}-${this.layoutThemeColor}`,
                `mdc-${this.themeMode}-${this.themeColor}`,
                'theme-link',
                () => {}
            );
        },
        changeThemeMode(mode) {
            this.themeMode = mode;
            this.changeTheme();
            this.$emit('layout-mode-change', mode);
        },
        changeThemeColor(color) {
            this.themeColor = color;
            this.changeTheme();
            this.$emit('layout-color-change', color);
        }
    },
    computed: {
        containerClass() {
            return ['layout-config', {'layout-config-active': this.active}];
        },
        rippleActive() {
            return this.$primevue.config.ripple;
        },
        inputStyle() {
            return this.$appState.inputStyle;
        }
    }
}
</script>

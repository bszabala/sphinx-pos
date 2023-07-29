<template>
    <div :class="containerClass" @click="onWrapperClick">
        <AppTopBar @menu-toggle="onMenuToggle"/>
        <div class="layout-sidebar" @click="onSidebarClick">
            <AppMenu :model="menu" @menuitem-click="onMenuItemClick"/>
        </div>

        <div class="layout-main-container">
            <div class="layout-main">
                <slot/>
            </div>
            <AppFooter/>
        </div>

        <AppConfig
            :layoutMode="layoutMode" :layoutThemeMode="layoutThemeMode" :layoutThemeColor="layoutThemeColor"
            @layout-change="onLayoutChange"
            @layout-mode-change="onLayoutThemeModeChange"
            @layout-color-change="onLayoutThemeColorChange" />
        <transition name="layout-mask">
            <div class="layout-mask p-component-overlay" v-if="mobileMenuActive"></div>
        </transition>
        <Toast/>
    </div>
</template>

<script>
import AppConfig from '../Components/AppConfig.vue';
import AppTopBar from '../Components/AppTopbar.vue';
import AppMenu from '../Components/AppMenu.vue';
import AppFooter from '../Components/AppFooter.vue';
import Toast from 'primevue/toast';

export default {
    data() {
        return {
            layoutMode: 'static',
            layoutThemeMode: 'light',
            layoutThemeColor: 'indigo',
            staticMenuInactive: false,
            overlayMenuActive: false,
            mobileMenuActive: false,
            menu: [
                {
                    label: 'Menu',
                    items: [
                        { label: 'Dashboard', icon: 'pi pi-fw pi-home', to: this.route('dashboard') }
                    ]
                },
                {
                    label: 'Manage',
                    items: [
                        { label: 'Categories', icon: 'pi pi-fw pi-th-large', to: this.route('users') },
                        { label: 'Products', icon: 'pi pi-fw pi-tags', to: this.route('roles') },
                        { label: 'Orders', icon: 'pi pi-fw pi-shopping-cart', to: this.route('permissions') },
                        { label: 'Reports', icon: 'pi pi-fw pi-print', to: this.route('permissions') },
                    ]
                },
                {
                    label: 'Settings',
                    items: [
                        {
                            label: 'Account',
                            icon: 'pi pi-fw pi-cog',
                            items: [
                                { label: 'Users', icon: 'pi pi-fw pi-users', to: this.route('users') },
                                { label: 'Roles', icon: 'pi pi-fw pi-sitemap', to: this.route('roles') },
                                { label: 'Permissions', icon: 'pi pi-fw pi-lock', to: this.route('permissions') }
                            ]
                        },
                        {
                            label: 'Sign out', icon: 'pi pi-fw pi-sign-out', command: () => {
                                this.$inertia.post(this.route('logout'))
                            },
                        }
                    ]
                }
            ]
        }
    },
    watch: {
        $route() {
            this.menuActive = false;
            this.$toast.removeAllGroups();
        }
    },
    mounted() {
        this.$nextTick(() => {
            this.onload();
        });
    },
    methods: {
        onload() {
            document.documentElement.style.fontSize = 13 + 'px';
        },
        onWrapperClick() {
            if (!this.menuClick) {
                this.overlayMenuActive = false;
                this.mobileMenuActive = false;
            }

            this.menuClick = false;
        },
        onMenuToggle(event) {
            this.menuClick = true;

            if (this.isDesktop()) {
                if (this.layoutMode === 'overlay') {
                    if (this.mobileMenuActive === true) {
                        this.overlayMenuActive = true;
                    }

                    this.overlayMenuActive = !this.overlayMenuActive;
                    this.mobileMenuActive = false;
                } else if (this.layoutMode === 'static') {
                    this.staticMenuInactive = !this.staticMenuInactive;
                }
            } else {
                this.mobileMenuActive = !this.mobileMenuActive;
            }

            event.preventDefault();
        },
        onSidebarClick() {
            this.menuClick = true;
        },
        onMenuItemClick(event) {
            if (event.item && !event.item.items) {
                this.overlayMenuActive = false;
                this.mobileMenuActive = false;
            }
        },
        onLayoutChange(layoutMode) {
            this.layoutMode = layoutMode;
        },
        onLayoutThemeModeChange(layoutThemeMode) {
            this.layoutThemeMode = layoutThemeMode;
        },
        onLayoutThemeColorChange(layoutThemeColor) {
            this.layoutThemeColor = layoutThemeColor;
        },
        addClass(element, className) {
            if (element.classList)
                element.classList.add(className);
            else
                element.className += ' ' + className;
        },
        removeClass(element, className) {
            if (element.classList)
                element.classList.remove(className);
            else
                element.className = element.className.replace(new RegExp('(^|\\b)' + className.split(' ').join('|') + '(\\b|$)', 'gi'), ' ');
        },
        isDesktop() {
            return window.innerWidth >= 992;
        },
        isSidebarVisible() {
            if (this.isDesktop()) {
                if (this.layoutMode === 'static')
                    return !this.staticMenuInactive;
                else if (this.layoutMode === 'overlay')
                    return this.overlayMenuActive;
            }

            return true;
        }
    },
    computed: {
        containerClass() {
            return [
                'layout-wrapper',
                {
                    'layout-overlay': this.layoutMode === 'overlay',
                    'layout-static': this.layoutMode === 'static',
                    'layout-static-sidebar-inactive': this.staticMenuInactive && this.layoutMode === 'static',
                    'layout-overlay-sidebar-active': this.overlayMenuActive && this.layoutMode === 'overlay',
                    'layout-mobile-sidebar-active': this.mobileMenuActive,
                    'p-input-filled': this.$primevue.config.inputStyle === 'filled',
                    'p-ripple-disabled': this.$primevue.config.ripple === false,
                }
            ];
        },
        logo() {
            return (this.layoutThemeMode === 'dark') ? "images/logo-white.svg" : "images/logo.svg";
        }
    },
    beforeUpdate() {
        if (this.mobileMenuActive)
            this.addClass(document.body, 'body-overflow-hidden');
        else
            this.removeClass(document.body, 'body-overflow-hidden');
    },
    components: {
        AppConfig,
        AppTopBar,
        AppMenu,
        AppFooter,
        Toast
    }
}
</script>

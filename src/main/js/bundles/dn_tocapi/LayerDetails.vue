<template>
    <v-list-tile
        :disabled="disabled">
        <v-list-tile-action
            @click.prevent.stop>
            <v-btn
                icon
                @click="item.visible = !item.visible">
                <v-icon :class="{'success--text': visible}">{{ visible ? config.visibleIconClass : config.invisibleIconClass }}</v-icon>
            </v-btn>
        </v-list-tile-action>
        <v-list-tile-action
            v-if="config.showLegend && $root.getLegend(item.layer.url)"
            @click="item.visible = !item.visible">
            <img :src="$root.getLegend(item.layer.url)">
        </v-list-tile-action>
        <v-list-tile-content
            @click.prevent.stop
            @click="item.visible = !item.visible"
        >
            <v-list-tile-title v-text="item.title"/>
        </v-list-tile-content>
        <v-list-tile-action
            v-if="showMenu"
            @click.prevent.stop
        >
            <v-menu
                :close-on-content-click="false"
                :close-on-click="true"
                v-model="menuOpen"
                bottom
                left
                max-width="300"
                min-width="300"
                nudge-top="42"
                nudge-right="4"
                offset-y
                transition="slide-y-transition"
                content-class="vue-toc__item-menu">
                <v-btn
                    slot="activator"
                    icon>
                    <v-icon>more_vert</v-icon>
                </v-btn>
                <layer-menu
                    :i18n="i18n"
                    :custom-layer-actions="customLayerActions"
                    :item="item"
                    @close-menu="menuOpen=false"/>
            </v-menu>
        </v-list-tile-action>
    </v-list-tile>
</template>
<script>
    import LayerMenu from "./LayerMenu.vue";

    export default {
        name: "layer-details",
        components: {
            "layer-menu": LayerMenu
        },
        props: [
            "bus",
            "item",
            "i18n",
            "config",
            "disabled",
            "customLayerActions"
        ],
        data: function () {
            let item = this.item;
            let layer = item.layer;
            let loaded = layer.loaded === undefined ? true : layer.loaded;
            let visible = item.visible;
            let initialVisible = item.initialVisible;
            return {
                menuOpen: false,
                loaded,
                visible,
                initialVisible: initialVisible,
                watchHandles: [],
                showMenu:  this.menuVisible() && loaded
            }
        },
        watch: {
            loaded: function () {
                this.showMenu = this.menuVisible() && this.loaded
            },
            visible: function () {
                this.showMenu = this.menuVisible() && this.loaded
            }
        },
        beforeMount: function () {
            this.watchHandles.push(this.item.watch("visible", visible => this.visible = visible));
            this.watchHandles.push(this.item.layer.watch("loaded", loaded => this.loaded = loaded));
            this.bus.$on('reset', this.reset);
        },
        beforeDestroy: function () {
            this.watchHandles.forEach(handle => handle.remove());
            this.bus.$off('reset', this.reset);
        },
        methods: {
            menuVisible: function () {
                let hasActions = this.customLayerActions.some(action =>
                    action.methods.displayActionForItem(this.item)
                );
                return hasActions && this.config.showLayerMenu;

            },
            reset: function () {
                this.visible = this.item.visible = this.initialVisible;
            }
        }
    };
</script>
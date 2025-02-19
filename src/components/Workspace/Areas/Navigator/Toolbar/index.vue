<!-- SPDX-License-Identifier: GPL-3.0-or-later
License: GNU GPLv3 or later. See the license file in the project root for more information.
Copyright © 2021 - present Aleksey Hoffman. All rights reserved.
-->

<template>
  <div class="workspace-area__toolbar">
    <component
      :is="item.component"
      v-for="(item, index) in toolbarComponents"
      :key="'workspace-toolbar-component-' + index"
      :icon="item.icon"
      :icon-size="item.iconSize"
      :icon-class="item.iconClass"
      :tooltip="item.tooltip"
      :on-click="item.onClick"
      :route-name="item.routeName"
    />
  </div>
</template>

<script>
import {mapFields} from 'vuex-map-fields'
import Button from '@/components/Button/index.vue'

export default {
  computed: {
    ...mapFields({
      currentDir: 'navigatorView.currentDir',
    }),
    toolbarComponents () {
      return [
        {
          component: Button,
          icon: 'mdi-menu-down',
          iconSize: '32px',
          iconClass: 'action-toolbar__icon',
          tooltip: 'Current directory context menu',
          onClick: (event) => {this.toggleCurrentDirContextMenu(event)},
        },
        {
          component: Button,
          icon: 'mdi-arrow-left',
          iconSize: '20px',
          iconClass: 'action-toolbar__icon',
          tooltip: 'History | open previous directory',
          onClick: () => {this.$store.dispatch('LOAD_PREVIOUS_HISTORY_PATH')},
        },
        {
          component: Button,
          icon: 'mdi-arrow-right',
          iconSize: '20px',
          iconClass: 'action-toolbar__icon',
          tooltip: 'History | open next directory',
          onClick: () => {this.$store.dispatch('LOAD_NEXT_HISTORY_PATH')},
        },
        {
          component: Button,
          icon: 'mdi-arrow-up',
          iconSize: '20px',
          iconClass: 'action-toolbar__icon',
          tooltip: 'Go up directory',
          onClick: () => {this.$store.dispatch('GO_UP_DIRECTORY')},
        },
        {
          component: Button,
          icon: 'mdi-refresh',
          iconSize: '20px',
          iconClass: 'action-toolbar__icon',
          tooltip: 'Reload current directory',
          onClick: () => {
            this.$store.dispatch('RELOAD_DIR', {
              scrollTop: false,
              emitNotification: true,
            })
          },
        },
        {
          component: 'address-bar',
        },
        {
          component: 'v-spacer',
        },
        {
          component: 'filter-field',
          routeName: 'navigator',
        },
      ]
    },
  },
  methods: {
    toggleCurrentDirContextMenu (payload) {
      this.$store.dispatch('DESELECT_ALL_DIR_ITEMS')
      this.$store.dispatch('ADD_TO_SELECTED_DIR_ITEMS', this.currentDir)
        .then(() => {
          this.$store.dispatch('SET_CONTEXT_MENU', {
            value: 'toggle',
            x: payload.clientX,
            y: payload.clientY,
          })
        })
    },
  },
}
</script>

<style scoped>
.workspace-area__toolbar {
  display: flex;
  align-items: center;
  gap: var(--toolbar-item-gap);
  height: var(--workspace-area-toolbar-height);
  margin: 0px;
  margin-bottom: 0px;
  padding: 0 8px;
  border-bottom: 0px solid var(--dir-item-card-border-3);
  border-color: var(--dir-item-card-border-3);
  border-width: 0 0 thin 0;
}
</style>
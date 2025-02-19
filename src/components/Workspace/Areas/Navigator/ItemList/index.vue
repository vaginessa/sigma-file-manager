<!-- SPDX-License-Identifier: GPL-3.0-or-later
License: GNU GPLv3 or later. See the license file in the project root for more information.
Copyright © 2021 - present Aleksey Hoffman. All rights reserved.
-->

<template>
  <div class="workspace__area-content">
    <WorkspaceAreaLoader />
    <div
      v-show="dirItemsInfoIsPartiallyFetched || dirItemsInfoIsFetched"
      class="fade-in-200ms"
    >
      <VirtualList
        v-if="navigatorLayout === 'list'"
        layout="list"
        :items="formattedDirItems"
      />
      <VirtualList
        v-if="navigatorLayout === 'grid'"
        layout="grid"
        :items="formattedDirItemsRows"
      />
    </div>
  </div>
</template>

<script>
import {mapFields} from 'vuex-map-fields'
import {mapState} from 'vuex'
import itemFilter from '@/utils/itemFilter'
import VirtualList from '@/components/VirtualList/index.vue'
import WorkspaceAreaLoader from '../Loader/index.vue'

const lodash = require('@/utils/lodash.min.js')

export default {
  components: {
    WorkspaceAreaLoader,
    VirtualList,
  },
  computed: {
    ...mapState({
      navigatorLayout: state => state.storageData.settings.navigatorLayout,
      navigatorLayoutItemHeight: state => state.storageData.settings.navigatorLayoutItemHeight,
      groupDirItems: state => state.storageData.settings.groupDirItems,
    }),
    ...mapFields({
      windowSize: 'windowSize',
      dirItems: 'navigatorView.dirItems',
      filteredDirItems: 'navigatorView.filteredDirItems',
      navigatorViewInfoPanel: 'storageData.settings.infoPanels.navigatorView',
      navigatorShowHiddenDirItems: 'storageData.settings.navigator.showHiddenDirItems',
      filterQuery: 'filterField.view.navigator.query',
      filterOptions: 'filterField.view.navigator.options',
      dirItemsInfoIsPartiallyFetched: 'navigatorView.dirItemsInfoIsPartiallyFetched',
      dirItemsInfoIsFetched: 'navigatorView.dirItemsInfoIsFetched',
      showDirItemKindDividers: 'storageData.settings.navigator.showDirItemKindDividers',
    }),
    contentAreaHeight () {
      const contentAreaHeight = this.windowSize.y -
       parseInt(this.$utils.getCSSVar('--window-toolbar-height')) -
       parseInt(this.$utils.getCSSVar('--action-toolbar-height')) -
       parseInt(this.$utils.getCSSVar('--workspace-area-toolbar-height'))
      return contentAreaHeight
    },
    formattedDirItemsRows () {
      // console.log('formattedDirItemsRows', this.dirItems)
      // if (this.dirItems.length === 0) {return []}

      let results = []
      let data = {
        gridColumnAmount: null,
        directoryRowsFormatted: [],
        fileRowsFormatted: [],
      }
      const dividerMarginBottom = 8
      const dirItemMarginBottom = 24
      const navMenuWidth = 64
      const itemMinWidth = 280
      const infoPanelWidth = 280
      const gapSize = 24
      const infoPanelCurrentWidth = this.navigatorViewInfoPanel.value ? infoPanelWidth : 0
      const container = document.querySelector('.virtual-list__root')
      let containerWidth = this.windowSize.x - navMenuWidth - infoPanelCurrentWidth

      // Update the container element width when it's loaded
      if (container !== null) {
        // Get the container padding so the item fit is calculated properly
        // NOTE: specifying padding for the "container.firstChild" instead of "container"
        // to avoid item card clipping when it's hovered (scaled).
        containerWidth = this.$utils.getNodeContentWidth(container.firstChild)
      }

      // TODO: Refactor: move all properties to data object
      data.gridColumnAmount = this.getGridColumnAmount({containerWidth, itemMinWidth})

      const gapAmount = data.gridColumnAmount - 1
      const gap = gapSize / data.gridColumnAmount
      const chunkItemsAmountWithIncludedGap = Math.floor(containerWidth / (itemMinWidth + (gap * gapAmount)))
      const imageFilesDirItems = this.imageFilesDirItems
      const videoFilesDirItems = this.videoFilesDirItems
      const otherFilesDirItems = this.otherFilesDirItems
      const fileDirItemsGrouped = [...imageFilesDirItems, ...videoFilesDirItems, ...otherFilesDirItems]
      const directoryDirItemsAsRows = lodash.chunk(
        this.directoryDirItems,
        chunkItemsAmountWithIncludedGap,
      )
      const fileDirItemsAsRows = lodash.chunk(
        this.fileDirItems,
        chunkItemsAmountWithIncludedGap,
      )
      const fileDirItemsAsRowsGrouped = lodash.chunk(
        fileDirItemsGrouped,
        chunkItemsAmountWithIncludedGap,
      )
      const topSpacer = [{
        isSpacer: true,
        path: 'top-spacer',
        type: 'top-spacer',
        height: this.showDirItemKindDividers ? 2 : 16,
        marginBottom: 0,
      }]
      const bottomSpacer = [{
        isSpacer: true,
        path: 'bottom-spacer',
        type: 'bottom-spacer',
        height: this.showDirItemKindDividers ? 2 : 16,
        marginBottom: 0,
      }]
      data.directoryRowsFormatted = directoryDirItemsAsRows.map(row => {
        return {
          type: 'directory-row',
          height: 64 + dirItemMarginBottom,
          marginBottom: dirItemMarginBottom,
          items: row,
        }
      })
      data.fileRowsFormatted = fileDirItemsAsRows.map(row => {
        return {
          type: 'file-row',
          height: 158 + dirItemMarginBottom,
          marginBottom: dirItemMarginBottom,
          items: row,
        }
      })
      const fileDirItemsAsRowsFormattedGrouped = fileDirItemsAsRowsGrouped.map(row => {
        return {
          type: 'file-row',
          height: 158 + dirItemMarginBottom,
          marginBottom: dirItemMarginBottom,
          items: row,
        }
      })
      const directoryDivider = this.directoryDirItems.length > 0
        ? [{
            isDivider: true,
            title: this.getDirItemGroupTitle('directory'),
            path: 'directory-divider',
            type: 'directory-divider',
            height: 36 + dividerMarginBottom,
            marginBottom: dividerMarginBottom,
          }]
        : []
      const fileDivider = this.fileDirItems.length > 0
        ? [{
            isDivider: true,
            title: this.getDirItemGroupTitle('file'),
            path: 'file-divider',
            type: 'file-divider',
            height: 36 + dividerMarginBottom,
            marginBottom: dividerMarginBottom,
          }]
        : []
      if (this.groupDirItems) {
        results = [
          ...topSpacer,
          ...directoryDivider,
          ...data.directoryRowsFormatted,
          ...fileDivider,
          ...fileDirItemsAsRowsFormattedGrouped,
          ...bottomSpacer,
        ]
      }
      else if (this.showDirItemKindDividers) {
        results = [
          ...topSpacer,
          ...directoryDivider,
          ...data.directoryRowsFormatted,
          ...fileDivider,
          ...data.fileRowsFormatted,
          ...bottomSpacer,
        ]
      }
      else {
        results = [
          ...topSpacer,
          ...data.directoryRowsFormatted,
          ...data.fileRowsFormatted,
          ...bottomSpacer,
        ]
      }
      // Add indexes
      results = results.map((item, index) => {
        item.positionIndex = index
        return item
      })
      // Set 'dirItemPositionIndex' property for dirItems
      const directoryDirItems = directoryDirItemsAsRows.flat()
      const fileDirItems = fileDirItemsAsRows.flat()
      let allDirItems = directoryDirItems.concat(fileDirItems)
      allDirItems = allDirItems.map((item, index) => {
        item.dirItemPositionIndex = index
        return item
      })
      this.setNavigatorViewInfo(data)
      return results
    },
    formattedDirItems () {
      // console.log('formattedDirItems', this.dirItems)
      // if (this.dirItems.length === 0) {return []}
      console.time('time::formattedDirItems')
      let results = []
      const directoryDivider = this.directoryDirItems.length > 0
        ? [{
            isDivider: true,
            title: this.getDirItemGroupTitle('directory'),
            path: 'directory-divider',
            type: 'directory-divider',
            height: 36,
          }]
        : []
      const fileDivider = this.fileDirItems.length > 0
        ? [{
            isDivider: true,
            title: this.getDirItemGroupTitle('file'),
            path: 'file-divider',
            type: 'file-divider',
            height: 36,
          }]
        : []
      const directoryDirItems = this.directoryDirItems
      const fileDirItems = this.fileDirItems
      const imageFilesDirItems = this.imageFilesDirItems
      const videoFilesDirItems = this.videoFilesDirItems
      const otherFilesDirItems = this.otherFilesDirItems
      const topSpacer = [{
        isSpacer: true,
        path: 'top-spacer',
        type: 'top-spacer',
        height: 2,
        marginBottom: 0,
      }]
      const bottomSpacer = [{
        isSpacer: true,
        path: 'bottom-spacer',
        type: 'bottom-spacer',
        height: 12,
        marginBottom: 0,
      }]
      if (this.groupDirItems) {
        results = [
          ...directoryDivider,
          ...directoryDirItems,
          ...fileDivider,
          ...imageFilesDirItems,
          ...videoFilesDirItems,
          ...otherFilesDirItems,
        ]
      }
      else if (this.showDirItemKindDividers) {
        results = [
          ...topSpacer,
          ...directoryDivider,
          ...directoryDirItems,
          ...fileDivider,
          ...fileDirItems,
          ...bottomSpacer,
        ]
      }
      else {
        results = [
          ...topSpacer,
          ...directoryDirItems,
          ...fileDirItems,
          ...bottomSpacer,
        ]
      }
      // Add indexes
      results = results.map((item, index) => {
        item.positionIndex = index
        return item
      })
      // Set 'dirItemPositionIndex' property for dirItems
      let allDirItems = directoryDirItems.concat(fileDirItems)
      allDirItems = allDirItems.map((item, index) => {
        item.dirItemPositionIndex = index
        return item
      })
      console.timeEnd('time::formattedDirItems')
      return results
    },
    directoryDirItems () {
      return [...this.getItemsMatchingFilter(this.dirItems).filter(item => {
        return item.type === 'directory' || item.type === 'directory-symlink'
      })]
    },
    fileDirItems () {
      return [...this.getItemsMatchingFilter(this.dirItems).filter(item => {
        return item.type === 'file' || item.type === 'file-symlink'
      })]
    },
    imageFilesDirItems () {
      return [...this.fileDirItems.filter(item => {
        return item.mime.mimeDescription === 'image'
      })]
    },
    videoFilesDirItems () {
      return [...this.fileDirItems.filter(item => {
        return item.mime.mimeDescription === 'video'
      })]
    },
    otherFilesDirItems () {
      return [...this.fileDirItems.filter(item => {
        return !['image', 'video'].includes(item.mime.mimeDescription)
      })]
    },
    dirItemsVisualPositioning () {
      const indexData = [...this.directoryDirItems, ...this.fileDirItems]
      return indexData
    },
  },
  methods: {
    setNavigatorViewInfo (data) {
      this.$store.dispatch('SET', {
        key: 'navigatorView.info',
        value: data,
      })
    },
    getGridColumnAmount (params) {
      return Math.floor(params.containerWidth / params.itemMinWidth)
    },
    getItemsMatchingFilter (items) {
      this.filteredDirItems = itemFilter({
        filterQuery: this.filterQuery,
        items,
        filterHiddenItems: !this.navigatorShowHiddenDirItems,
        filterProperties: this.$store.state.filterField.view.navigator.filterProperties,
        filterQueryOptions: this.$store.state.filterField.view.navigator.options,
      })
      return this.filteredDirItems
    },
    getDirItemGroupTitleDescription (itemCount) {
      const itemWord = this.$localizeUtils.pluralize(itemCount, 'item')
      return this.dirItemsInfoIsFetched ? `${itemCount} ${itemWord}` : `Loading ${itemWord}`
    },
    getDirItemGroupTitle (type) {
      if (type === 'directory') {
        const itemCount = this.directoryDirItems.length
        return `Directories | ${this.getDirItemGroupTitleDescription(itemCount)}`
      }
      else if (type === 'file') {
        const itemCount = this.fileDirItems.length
        return `Files | ${this.getDirItemGroupTitleDescription(itemCount)}`
      }
      else if (type === 'image-files') {
        const itemCount = this.imageFilesDirItems.length
        return `Images | ${this.getDirItemGroupTitleDescription(itemCount)}`
      }
      else if (type === 'video-files') {
        const itemCount = this.videoFilesDirItems.length
        return `Videos | ${this.getDirItemGroupTitleDescription(itemCount)}`
      }
      else if (type === 'other-files') {
        const itemCount = this.otherFilesDirItems.length
        return `Other | ${this.getDirItemGroupTitleDescription(itemCount)}`
      }
    },
  },
}
</script>

<style>
.workspace__area-content {
  height: 100%;
  flex: 1 1 auto;
}
</style>
<!-- SPDX-License-Identifier: GPL-3.0-or-later
License: GNU GPLv3 or later. See the license file in the project root for more information.
Copyright © 2021 - present Aleksey Hoffman. All rights reserved.
-->

<template>
  <div
    class="item-card__container"
    :lines="lines"
  >
    <slot></slot>
  </div>
</template>

<script>
export default {
  props: {
    lines: {
      type: Number,
      default: 1,
    },
  },
}
</script>

<style>
.item-card__container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  grid-auto-rows: 48px;
  gap: 16px;
}

.item-card__container[lines="2"] {
  grid-auto-rows: 64px;
}
</style>
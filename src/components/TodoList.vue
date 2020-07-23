<template>
  <v-list>
    <v-list-item
      v-for="(item, index) in items"
      :key="index"
      link
      class="item h-15"
      @click="$emit('itemClick',item)"
    >
      <v-list-item-action>
        <v-progress-circular
          indeterminate
          v-if="updating === item.todoId"
          :size="24"
          color="primary"
        />
        <v-checkbox :input-value="item.complete" color="primary" v-else />
      </v-list-item-action>

      <v-list-item-content>
        <v-list-item-title v-text="item.name" />
      </v-list-item-content>

      <v-list-item-action class="invisible delete-btn">
        <v-progress-circular indeterminate v-if="deleting === item.todoId" :size="24" color="grey" />
        <v-btn icon @click.stop="$emit('itemDelete', item)" v-else>
          <v-icon color="grey lighten-1">delete_outline</v-icon>
        </v-btn>
      </v-list-item-action>
    </v-list-item>
  </v-list>
</template>

<script>
export default {
  props: ["items", "updating", "deleting"],
};
</script>

<style lang="sass" scoped>
.item:hover
  .delete-btn
    visibility: visible
</style>
<template>
  <template v-for="genre in genres" :key="genre.itemId">
    <div v-if="!genre.isItem && !hide_group_title" class="mt-6 mb-5 py-2">
      <div class="media-content is-clipped">
        <span
          :id="'index_' + genre.groupKey"
          class="tag is-info is-light is-small has-text-weight-bold"
          v-text="genre.groupKey"
        />
      </div>
    </div>
    <div v-else-if="genre.isItem" class="media" @click="open_genre(genre.item)">
      <div class="media-content fd-has-action is-clipped">
        <h1 class="title is-6" v-text="genre.item.name" />
      </div>
      <div class="media-right">
        <a @click.prevent.stop="open_dialog(genre.item)">
          <span class="icon has-text-dark"
            ><mdicon name="dots-vertical" size="16"
          /></span>
        </a>
      </div>
    </div>
  </template>
  <teleport to="#app">
    <modal-dialog-genre
      :show="show_details_modal"
      :genre="selected_genre"
      @close="show_details_modal = false"
    />
  </teleport>
</template>

<script>
import ModalDialogGenre from '@/components/ModalDialogGenre.vue'

export default {
  name: 'ListGenres',
  components: { ModalDialogGenre },

  props: ['genres', 'media_kind', 'hide_group_title'],

  data() {
    return {
      show_details_modal: false,
      selected_genre: {}
    }
  },

  computed: {
    media_kind_resolved: function () {
      return this.media_kind ? this.media_kind : this.selected_genre.media_kind
    }
  },

  methods: {
    open_genre: function (genre) {
      this.$router.push({ name: 'Genre', params: { genre: genre.name } })
    },

    open_dialog: function (genre) {
      this.selected_genre = genre
      this.show_details_modal = true
    }
  }
}
</script>

<style></style>

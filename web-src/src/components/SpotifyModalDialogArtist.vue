<template>
  <div>
    <transition name="fade">
      <div v-if="show" class="modal is-active">
        <div class="modal-background" @click="$emit('close')" />
        <div class="modal-content fd-modal-card">
          <div class="card">
            <div class="card-content">
              <p class="title is-4">
                <a
                  class="has-text-link"
                  @click="open_artist"
                  v-text="artist.name"
                />
              </p>
              <div class="content is-small">
                <p>
                  <span
                    class="heading"
                    v-text="$t('dialog.spotify.artist.popularity')"
                  />
                  <span
                    class="title is-6"
                    v-text="
                      [artist.popularity, artist.followers.total].join(' / ')
                    "
                  />
                </p>
                <p>
                  <span
                    class="heading"
                    v-text="$t('dialog.spotify.artist.genres')"
                  />
                  <span class="title is-6" v-text="artist.genres.join(', ')" />
                </p>
              </div>
            </div>
            <footer class="card-footer">
              <a class="card-footer-item has-text-dark" @click="queue_add">
                <span class="icon"
                  ><mdicon name="playlist-plus" size="16"
                /></span>
                <span
                  class="is-size-7"
                  v-text="$t('dialog.spotify.artist.add')"
                />
              </a>
              <a class="card-footer-item has-text-dark" @click="queue_add_next">
                <span class="icon"
                  ><mdicon name="playlist-play" size="16"
                /></span>
                <span
                  class="is-size-7"
                  v-text="$t('dialog.spotify.artist.add-next')"
                />
              </a>
              <a class="card-footer-item has-text-dark" @click="play">
                <span class="icon"><mdicon name="play" size="16" /></span>
                <span
                  class="is-size-7"
                  v-text="$t('dialog.spotify.artist.play')"
                />
              </a>
            </footer>
          </div>
        </div>
        <button
          class="modal-close is-large"
          aria-label="close"
          @click="$emit('close')"
        />
      </div>
    </transition>
  </div>
</template>

<script>
import webapi from '@/webapi'

export default {
  name: 'SpotifyModalDialogArtist',
  props: ['show', 'artist'],
  emits: ['close'],

  methods: {
    play: function () {
      this.$emit('close')
      webapi.player_play_uri(this.artist.uri, false)
    },

    queue_add: function () {
      this.$emit('close')
      webapi.queue_add(this.artist.uri)
    },

    queue_add_next: function () {
      this.$emit('close')
      webapi.queue_add_next(this.artist.uri)
    },

    open_artist: function () {
      this.$router.push({ path: '/music/spotify/artists/' + this.artist.id })
    }
  }
}
</script>

<style></style>

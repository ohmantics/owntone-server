<template>
  <div class="fd-page-with-tabs">
    <tabs-music />
    <!-- New Releases -->
    <content-with-heading>
      <template #heading-left>
        <p class="title is-4" v-text="$t('page.spotify.browse.new-releases')" />
      </template>
      <template #content>
        <spotify-list-item-album
          v-for="album in new_releases"
          :key="album.id"
          :album="album"
          @click="open_album(album)"
        >
          <template v-if="is_visible_artwork" #artwork>
            <p class="image is-64x64 fd-has-shadow fd-has-action">
              <cover-artwork
                :artwork_url="artwork_url(album)"
                :artist="album.artist"
                :album="album.name"
                :maxwidth="64"
                :maxheight="64"
              />
            </p>
          </template>
          <template #actions>
            <a @click.prevent.stop="open_album_dialog(album)">
              <span class="icon has-text-dark"
                ><mdicon name="dots-vertical" size="16"
              /></span>
            </a>
          </template>
        </spotify-list-item-album>
        <spotify-modal-dialog-album
          :show="show_album_details_modal"
          :album="selected_album"
          @close="show_album_details_modal = false"
        />
      </template>
      <template #footer>
        <nav class="level">
          <p class="level-item">
            <router-link
              to="/music/spotify/new-releases"
              class="button is-light is-small is-rounded"
              >{{ $t('page.spotify.browse.show-more') }}</router-link
            >
          </p>
        </nav>
      </template>
    </content-with-heading>
    <!-- Featured Playlists -->
    <content-with-heading>
      <template #heading-left>
        <p
          class="title is-4"
          v-text="$t('page.spotify.browse.featured-playlists')"
        />
      </template>
      <template #content>
        <spotify-list-item-playlist
          v-for="playlist in featured_playlists"
          :key="playlist.id"
          :playlist="playlist"
        >
          <template #actions>
            <a @click.prevent.stop="open_playlist_dialog(playlist)">
              <span class="icon has-text-dark"
                ><mdicon name="dots-vertical" size="16"
              /></span>
            </a>
          </template>
        </spotify-list-item-playlist>
        <spotify-modal-dialog-playlist
          :show="show_playlist_details_modal"
          :playlist="selected_playlist"
          @close="show_playlist_details_modal = false"
        />
      </template>
      <template #footer>
        <nav class="level">
          <p class="level-item">
            <router-link
              to="/music/spotify/featured-playlists"
              class="button is-light is-small is-rounded"
              >{{ $t('page.spotify.browse.show-more') }}</router-link
            >
          </p>
        </nav>
      </template>
    </content-with-heading>
  </div>
</template>

<script>
import ContentWithHeading from '@/templates/ContentWithHeading.vue'
import TabsMusic from '@/components/TabsMusic.vue'
import SpotifyListItemAlbum from '@/components/SpotifyListItemAlbum.vue'
import SpotifyListItemPlaylist from '@/components/SpotifyListItemPlaylist.vue'
import SpotifyModalDialogAlbum from '@/components/SpotifyModalDialogAlbum.vue'
import SpotifyModalDialogPlaylist from '@/components/SpotifyModalDialogPlaylist.vue'
import CoverArtwork from '@/components/CoverArtwork.vue'
import store from '@/store'
import * as types from '@/store/mutation_types'
import SpotifyWebApi from 'spotify-web-api-js'

const dataObject = {
  load: function (to) {
    if (
      store.state.spotify_new_releases.length > 0 &&
      store.state.spotify_featured_playlists.length > 0
    ) {
      return Promise.resolve()
    }

    const spotifyApi = new SpotifyWebApi()
    spotifyApi.setAccessToken(store.state.spotify.webapi_token)
    return Promise.all([
      spotifyApi.getNewReleases({
        country: store.state.spotify.webapi_country,
        limit: 50
      }),
      spotifyApi.getFeaturedPlaylists({
        country: store.state.spotify.webapi_country,
        limit: 50
      })
    ])
  },

  set: function (vm, response) {
    if (response) {
      store.commit(types.SPOTIFY_NEW_RELEASES, response[0].albums.items)
      store.commit(
        types.SPOTIFY_FEATURED_PLAYLISTS,
        response[1].playlists.items
      )
    }
  }
}

export default {
  name: 'SpotifyPageBrowse',
  components: {
    ContentWithHeading,
    TabsMusic,
    SpotifyListItemAlbum,
    SpotifyListItemPlaylist,
    SpotifyModalDialogAlbum,
    SpotifyModalDialogPlaylist,
    CoverArtwork
  },

  beforeRouteEnter(to, from, next) {
    dataObject.load(to).then((response) => {
      next((vm) => dataObject.set(vm, response))
    })
  },
  beforeRouteUpdate(to, from, next) {
    const vm = this
    dataObject.load(to).then((response) => {
      dataObject.set(vm, response)
      next()
    })
  },

  data() {
    return {
      show_album_details_modal: false,
      selected_album: {},

      show_playlist_details_modal: false,
      selected_playlist: {}
    }
  },

  computed: {
    new_releases() {
      return this.$store.state.spotify_new_releases.slice(0, 3)
    },

    featured_playlists() {
      return this.$store.state.spotify_featured_playlists.slice(0, 3)
    },

    is_visible_artwork() {
      return this.$store.getters.settings_option(
        'webinterface',
        'show_cover_artwork_in_album_lists'
      ).value
    }
  },

  methods: {
    open_album: function (album) {
      this.$router.push({ path: '/music/spotify/albums/' + album.id })
    },

    open_album_dialog: function (album) {
      this.selected_album = album
      this.show_album_details_modal = true
    },

    open_playlist_dialog: function (playlist) {
      this.selected_playlist = playlist
      this.show_playlist_details_modal = true
    },

    artwork_url: function (album) {
      if (album.images && album.images.length > 0) {
        return album.images[0].url
      }
      return ''
    }
  }
}
</script>

<style></style>

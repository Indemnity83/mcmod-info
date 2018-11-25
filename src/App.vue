<template>
    <div id="app">
        <input ref="mod" type="file" name="mod" @change="update">

        <pre v-highlightjs="info"><code class="JSON"></code></pre>
    </div>
</template>

<script>

import JSZip from "jszip";

export default {
    name: 'app',

    data() {
        return {
            info: null,
            files: [],
            errors: null,
        }
    },

    methods: {
        /**
         * Update the modpack's icon.
         */
        update(e) {
            e.preventDefault();

            let files = this.$refs.mod.files;

            if (!files.length) {
                return;
            }

            for (let i = 0; i < files.length; i++) {
                this.handleFile(files[i]);
            }
        },

        handleFile(file) {
            let self = this;

            JSZip.loadAsync(file)
                .then(function(zip) {
                    zip.file("mcmod.info").async("string")
                        .then(function(info) {
                            self.info = info;
                        });
                }, function (e) {
                    self.errors.append("Error reading " + file.name + ": " + e.message);
                });
        }
    }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

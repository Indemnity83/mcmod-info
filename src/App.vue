<template>
    <div id="app">
        <h1>Choose one or more jar files</h1>
        <button @click="clear">Clear Results</button>
        <input ref="mod" type="file" name="mod" @change="update" multiple>

        <div v-for="file in files" :key="file.name">
            <h2>{{ file.name }}</h2>
            <p>{{ file.error }}</p>
            <pre v-highlightjs="file.info" v-if="file.info"><code class="JSON"></code></pre>

        </div>
    </div>
</template>

<script>

import JSZip from "jszip";

export default {
    name: 'app',

    data() {
        return {
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

        clear() {
            this.files = [];
            this.$refs.mod.value = null;
        },

        handleFile(modFile) {
            let self = this;

            JSZip.loadAsync(modFile)
                .then(function(zip) {
                    let mcmod = zip.file("mcmod.info");

                    if(mcmod === null) {
                        let parsedFile = {
                            name: modFile.name,
                            error: "missing mcmod.info file",
                        };

                        self.files.push(parsedFile);
                        return;
                    }

                    mcmod.async("string")
                        .then(function(info) {
                            let parsedFile = {
                                name: modFile.name,
                                info: info,
                            };

                            self.files.push(parsedFile);
                        });
                }, function (e) {
                    self.errors.append("Error reading " + modFile.name + ": " + e.message);
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
  margin-top: 20px;
}
</style>

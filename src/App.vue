<template>
    <div id="app" class="container">

        <div class="alert alert-danger" v-if="errors">
            {{ errors }}
        </div>

        <div class="form-group d-flex">
            <div class="uploader mr-4">
                <input ref="mod" type="file" name="mod" @change="update" multiple class="uploader-control">
                <div class="btn btn-outline-dark">Select Mods</div>
            </div>

            <button class="btn btn-outline-dark" @click="clear">Clear Results</button>
        </div>

        <div class="card mb-4" v-for="file in files" :key="file.name">
            <div class="card-header">
                {{ file.name }}
            </div>
            <div
                v-if="file.error"
                class="mb-0 rounded-0 border-0 alert alert-danger"
                role="alert">
                {{ file.error }}
            </div>

            <pre class="mb-0 rounded-0" v-highlightjs="file.info" v-if="file.info"><code class="JSON"></code></pre>
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

            this.$refs.mod.value = null;
        },

        clear() {
            this.files = [];
            this.$refs.mod.value = null;
        },

        handleFile(modFile) {
            let self = this;

            JSZip.loadAsync(modFile)
                .then(function(zip) {

                    // Is it forge?
                    let forge = zip.file("version.json");
                    if(forge !== null) {
                        forge.async("string")
                            .then(function(info) {
                                let parsedFile = {
                                    name: modFile.name,
                                    info: info,
                                };

                                self.files.push(parsedFile);
                            });

                        return;
                    }

                    // Is it a forge mod?
                    let mcmod = zip.file("mcmod.info");

                    if(mcmod !== null) {
                        mcmod.async("string")
                            .then(function(info) {
                                let parsedFile = {
                                    name: modFile.name,
                                    info: info,
                                };

                                self.files.push(parsedFile);
                            });

                        return;
                    }

                    // ¯\_(ツ)_/¯
                    let parsedFile = {
                        name: modFile.name,
                        error: "Invalid file",
                    };

                    self.files.push(parsedFile);

                }, function (e) {
                    self.errors.append("Error reading " + modFile.name + ": " + e.message);
                });
        }
    }
}
</script>

<style>
#app {
  margin-top: 30px;
}

.uploader {
    position: relative;
    border-radius: 0.25rem;
    cursor: pointer !important;
}

.uploader-control {
    position: absolute;
    top: 0;
    bottom: 0;
    max-width: 100%;
    opacity: 0;
    z-index: 99;
}
</style>

<template>
    <div id="app" class="container">

        <div class="alert alert-danger" v-if="errors">
            {{ errors }}
        </div>

        <div class="form-group d-flex mb-4">
            <div class="uploader mr-4">
                <input ref="mod" type="file" name="mod" @change="update" multiple class="uploader-control">
                <div class="btn btn-outline-dark">Select Mods</div>
            </div>

            <button class="btn btn-outline-dark" @click="clear">Clear Results</button>
        </div>

        <div class="card card-default mb-4">
            <div class="card-header">Mods</div>
            <table class="table mb-0">
                <thead>
                <tr>
                    <th>Filename</th>
                    <th>Name</th>
                    <th>Mod ID</th>
                    <th>Version</th>
                    <th>&nbsp;</th>
                </tr>
                </thead>
                <tbody>
                <tr v-for="file in files" :key="file.name">
                    <td class="table-fit">
                        {{ file.name }}
                    </td>
                    <td class="table-fit">
                        {{ file.mod.name }}
                    </td>
                    <td class="table-fit">
                        {{ file.mod.modid }}
                    </td>
                    <td class="table-fit">
                        {{ file.mod.version }}
                    </td>
                    <td class="text-right">
                        <button class="btn btn-outline-dark btn-sm" @click="show = file">
                            {{ file.infoFile }}
                        </button>
                    </td>
                </tr>
                </tbody>
            </table>
        </div>

        <div class="card mb-4" v-if="show !== null">
            <div class="card-header">
                {{ show.name }}/{{ show.infoFile }}
            </div>
            <div v-if="show.error" class="mb-0 rounded-0 border-0 alert alert-danger">
                {{ show.error }}
            </div>
            <pre class="mb-0 rounded-0" v-highlightjs="show.info" v-if="show.info"><code class="JSON"></code></pre>
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
            show: null,
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
            this.show = null;
            this.$refs.mod.value = null;
        },

        handleFile(modFile) {
            let self = this;

            JSZip.loadAsync(modFile)
                .then(function(zip) {

                    let file = {
                        name: modFile.name,
                        infoFile: null,
                        info: null,
                        mod: {
                            name: null,
                            modid: null,
                            version: null,
                        },
                        error: null,
                    };

                    // Is it forge?
                    let forge = zip.file("version.json");
                    if(forge !== null) {

                        forge.async("string")
                        .then(function(versionJson) {
                            try {
                                let info = JSON.parse(versionJson);
                                file.infoFile = "version.json";
                                file.info = info;
                                file.mod.name = 'Minecraft Forge';
                                file.mod.version = info.id;
                                file.mod.modid = 'minecraftforge';
                            } catch (e) {
                                file.error = e.message;
                            }
                        });

                    }

                    // Is it a forge mod?
                    let mcmod = zip.file("mcmod.info");

                    if(mcmod !== null) {
                        mcmod.async("string")
                        .then(function(mcmodInfo) {
                            try {
                                let info = JSON.parse(mcmodInfo);
                                file.infoFile = "mcmod.info";
                                file.info = mcmodInfo;

                                let mod = {name: null, version: null, modid: null};
                                if(info.modListVersion === 2 || info.modInfoVersion === 2) {
                                    mod = info.modList[0];
                                } else {
                                    mod = info[0];
                                }

                                file.mod.name = mod.name;
                                file.mod.version = mod.version;
                                file.mod.modid = mod.modid;
                            } catch (e) {
                                file.error = e.message;
                            }
                        });
                    }

                    self.files.push(file);

                }, function (e) {
                    self.errors.append("Error reading " + modFile.name + ": " + e.message);
                });
        },
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

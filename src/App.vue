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
                    <th>&nbsp;</th>
                    <th>&nbsp;</th>
                    <th>Filename</th>
                    <th>Name</th>
                    <th>Mod ID</th>
                    <th>Version</th>
                    <th>&nbsp;</th>
                </tr>
                </thead>
                <tbody>
                <tr v-for="file in files" :key="file.name">
                    <td class="align-middle">
                        <i :class="[ file.errors.length > 0 ? 'text-danger fa-times' : 'text-success fa-check', 'fas']"></i>
                    </td>
                    <td>
                        <img class="img-thumbnail" :src="file.mod.logoData" />
                    </td>
                    <td class="align-middle">
                        {{ file.name }}
                    </td>
                    <td colspan="3" v-if="file.errors.length > 0" class="align-middle text-danger">
                        errors: {{ file.errors.join(', ') }}
                    </td>
                    <td v-if="file.errors.length === 0" class="align-middle">
                        {{ file.mod.name }}
                    </td>
                    <td v-if="file.errors.length === 0" class="align-middle">
                        {{ file.mod.modid }}
                    </td>
                    <td v-if="file.errors.length === 0" class="align-middle">
                        {{ file.mod.version }}
                    </td>
                    <td class="align-middle text-right text-nowrap">
                        <button class="btn btn-outline-dark btn-sm" @click="show = file" v-if="file.infoFile !== null">
                            {{ file.infoFile }}
                        </button>
                        <button class="ml-2 btn btn-outline-danger btn-sm" @click="submitIssue(file)" v-if="file.errors.length > 0">
                            Create GitHub Issue
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
                            logoData: null,
                        },
                        errors: [],
                    };

                    // Is it forge?
                    let forge = zip.file("version.json");
                    if(forge !== null) {

                        forge.async("string")
                        .then(function(versionJson) {
                            try {
                                file.infoFile = "version.json";
                                file.info = info;

                                let info = JSON.parse(versionJson);
                                file.mod.name = 'Minecraft Forge';
                                file.mod.version = info.id;
                                file.mod.modid = 'minecraftforge';
                            } catch (e) {
                                file.errors.push(e.toString());
                            }
                        });

                    }

                    // Is it a forge mod?
                    let mcmod = zip.file("mcmod.info");

                    if(mcmod !== null) {
                        mcmod.async("string")
                        .then(function(mcmodInfo) {
                            try {
                                file.infoFile = "mcmod.info";
                                file.info = mcmodInfo;

                                let info = JSON.parse(mcmodInfo);

                                let mod = {name: null, version: null, modid: null};
                                if(info.modListVersion === 2 || info.modInfoVersion === 2) {
                                    mod = info.modList[0];
                                } else {
                                    mod = info[0];
                                }

                                file.mod.name = mod.name;
                                file.mod.version = mod.version;
                                file.mod.modid = mod.modid;

                                if(mod.logoFile !== null && mod.logoFile !== undefined) {
                                    let logoFile = zip.file(mod.logoFile.substring(1));
                                    if(logoFile !== null) {
                                        logoFile.async("base64").then(function(logoData) {
                                            file.mod.logoData = "data:image/png;base64," + logoData
                                        });
                                    }
                                }
                            } catch (e) {
                                file.errors.push(e.toString());
                            }
                        });
                    }

                    self.files.push(file);

                }, function (e) {
                    self.errors.push("Error reading " + modFile.name + ": " + e.message);
                });
        },

        submitIssue(file) {
            let subject = encodeURIComponent('[Parse Fail] ' + file.name);
            let body =
                "### Parsed Data\n" +
                "**Info File:** `" + file.infoFile + "`\n" +
                "**name:** `" + file.mod.name + "`\n" +
                "**modid:** `" + file.mod.modid + "`\n" +
                "**version:** `" + file.mod.version + "`\n" +
                "\n" +
                "**errors:** \n" +
                "1. " + file.errors.join("\n1. ") + "\n" +
                "\n" +
                "**mcmod.info contents**\n" +
                "```json\n" +
                file.info + "\n" +
                "```"
            ;

            body = encodeURIComponent(body);

            window.open(`https://github.com/indemnity83/mcmod-info/issues/new?title=${subject}&body=${body}`)
        }
    }
}
</script>

<style>
#app {
  margin-top: 30px;
}

.img-thumbnail {
    border: none;
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

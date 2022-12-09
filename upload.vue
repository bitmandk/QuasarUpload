<script>
    export default {
        data() {
            return {
                allFiles: [],
                documentTypes: [
                    { value: 1, label: 'Type 1' },
                    { value: 2, label: 'Type 2' },
                ],
                caseGuid: '686296F3-EEC5-4331-8A32-C885815297F6'
            };
        },
        computed: {
            uploadEndpoint() {
                return `https://localhost/cases/${encodeURIComponent(this.caseGuid)}/documents/upload`;
            },
        },
        methods: {
            getFile(key) {
                return this.allFiles.find((f) => f.__key === key);
            },

            onAdd(files) {
                //preselect document with documentTypeId == 1
                const docType = this.documentTypes.find((d) => d.value === 1);
                this.allFiles = this.allFiles.concat(
                    files.map((v) => ({
                        ...v,
                        docType,
                        remarks: null
                    }))
                );
            },

            onRemove(files) {
                const keys = files.map((f) => f.__key);
                this.allFiles = this.allFiles.filter((v) => keys.indexof(v.__key) === -1);
            },

            formFields(files) {
                // get the file from list of files
                const key = files[0].__key;
                const file = this.allFiles.find((v) => v.__key === key);
                // assign the extra props to form data
                return [
                    { name: 'DocumentType', value: file.docType.value },
                    { name: 'Remarks', value: file.remarks },
                ];
            }
        }
    };
</script>
<template>
    <div>
        <q-uploader style="width: auto; max-width: 600px"
                    :url="uploadEndpoint"
                    label="Tilføj dokumenter"
                    multiple
                    :form-fields="formFields"
                    @added="onAdd"
                    @removed="onRemove">
            <template v-slot:list="scope">
                <q-list separator>
                    <template v-for="file in scope.files" :key="file.__key">
                        <q-item>
                            <q-item-section>
                                <q-item-label class="full-width ellipsis">
                                    {{
                  file.name
                                    }}
                                </q-item-label>
                                <q-item-label caption>Status: {{ file.__status }}</q-item-label>
                                <q-item-label caption>
                                    {{ file.__sizeLabel }} /
                                    {{ file.__progressLabel }}
                                </q-item-label>
                            </q-item-section>
                            <q-item-section v-if="file.__img && $q.screen.gt.xs" thumbnail>
                                <img :src="file.__img.src" />
                            </q-item-section>
                            <q-item-section top side>
                                <q-btn class="gt-xs"
                                       size="12px"
                                       flat
                                       dense
                                       round
                                       icon="delete"
                                       @click="scope.removeFile(file)">
                                    <q-tooltip>Fjern dokument fra upload kø</q-tooltip>
                                </q-btn>
                            </q-item-section>
                        </q-item>
                        <q-item>
                            <q-item-section>
                                <q-select v-model="getFile(file.__key).docType"
                                          square
                                          outlined
                                          dense
                                          label="Dokumenttype"
                                          :options="documentTypes"></q-select>
                            </q-item-section>
                            <q-item-section>
                                <q-input v-model="getFile(file.__key).remarks"
                                         clearable
                                         square
                                         outlined
                                         dense
                                         label="Remark"></q-input>
                            </q-item-section>
                        </q-item>
                    </template>
                </q-list>
            </template>
        </q-uploader>
    </div>
</template>

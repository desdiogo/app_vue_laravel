<template>
    <v-container>
        <v-data-table
            :headers="headers"
            :items="carros"
            sort-by="calories"
            class="elevation-1"
        >
            <template v-slot:top>
                <v-toolbar flat>
                    <v-toolbar-title>Carros</v-toolbar-title>
                    <v-divider class="mx-4" inset vertical></v-divider>
                    <v-spacer></v-spacer>
                    <v-dialog v-model="dialog" max-width="500px">
                        <template v-slot:activator="{ on, attrs }">
                            <v-btn
                                color="primary"
                                dark
                                class="mb-2"
                                v-bind="attrs"
                                v-on="on"
                            >
                                Novo Carro
                            </v-btn>
                        </template>
                        <v-card>
                            <v-card-title>
                                <span class="headline">{{ formTitle }}</span>
                            </v-card-title>

                            <v-card-text>
                                <v-container>
                                    <v-form
                                        ref="form"
                                        v-model="valid"
                                        lazy-validation
                                    >
                                        <v-text-field
                                            v-model="editedItem.nome"
                                            :counter="30"
                                            :rules="nameRules"
                                            label="Nome"
                                            required
                                        ></v-text-field>

                                        <v-row>
                                            <v-col cols="6" align="start">
                                                <v-subheader>
                                                    Selecione a Marca
                                                </v-subheader>
                                            </v-col>

                                            <v-col cols="6">
                                                <v-select
                                                    v-model="
                                                        editedItem.marca_id
                                                    "
                                                    :items="marcas"
                                                    item-text="nome"
                                                    item-value="id"
                                                    return-object
                                                    single-line
                                                ></v-select>
                                            </v-col>
                                        </v-row>

                                        <v-text-field
                                            v-model="editedItem.modelo"
                                            :counter="4"
                                            :rules="modeloRules"
                                            label="Modelo"
                                            required
                                            type="number"
                                        ></v-text-field>

                                        <v-text-field
                                            v-model="editedItem.ano"
                                            :counter="4"
                                            :rules="anoRules"
                                            label="Ano"
                                            required
                                            type="number"
                                        ></v-text-field>

                                        <v-btn
                                            :disabled="!valid"
                                            color="success"
                                            class="mr-4 capitalize"
                                            @click="validate"
                                        >
                                            {{
                                                editedIndex === -1
                                                    ? "Salvar"
                                                    : "Editar"
                                            }}
                                        </v-btn>
                                        <v-btn
                                            color="orange"
                                            class="mr-4 capitalize"
                                            @click="close"
                                        >
                                            Fechar
                                        </v-btn>
                                    </v-form>
                                </v-container>
                            </v-card-text>
                        </v-card>
                    </v-dialog>
                    <v-dialog v-model="dialogDelete" max-width="500px">
                        <v-card>
                            <v-card-title class="headline"
                                >Confirma a exclusão do registro?</v-card-title
                            >
                            <v-card-actions>
                                <v-spacer></v-spacer>
                                <v-btn
                                    color="blue darken-1"
                                    text
                                    @click="closeDelete"
                                    >Fechar</v-btn
                                >
                                <v-btn
                                    color="blue darken-1"
                                    text
                                    @click="deleteItemConfirm"
                                    >OK</v-btn
                                >
                                <v-spacer></v-spacer>
                            </v-card-actions>
                        </v-card>
                    </v-dialog>
                </v-toolbar>
            </template>
            <template v-slot:[`item.actions`]="{ item }">
                <v-icon small class="mr-2" @click="editItem(item)">
                    mdi-pencil
                </v-icon>
                <v-icon small @click="deleteItem(item)">
                    mdi-delete
                </v-icon>
            </template>
            <template v-slot:no-data> </template>
        </v-data-table>
        <v-overlay :value="overlay" align="center">
            <v-progress-circular
                :size="50"
                color="primary"
                indeterminate
            ></v-progress-circular>
            <p>{{ messageOverlay }}</p>
        </v-overlay>
        <v-dialog v-model="dialog1" width="500">
            <v-card>
                <v-card-text>
                    {{ message }}
                </v-card-text>

                <v-divider></v-divider>

                <v-card-actions>
                    <v-btn color="primary" text @click="dialog1 = false">
                        Ok
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </v-container>
</template>

<script>
import axios from "axios";

export default {
    data: () => ({
        dialog: false,
        dialogDelete: false,
        headers: [
            {
                text: "Nome",
                align: "start",
                value: "nome",
            },
            { text: "Marca", value: "marca.nome" },
            { text: "Modelo", value: "modelo" },
            { text: "Ano", value: "ano" },
            { text: "Ações", value: "actions", sortable: false },
        ],
        editedIndex: -1,
        editedItem: {
            id: null,
            nome: "",
            marca_id: "",
            modelo: "",
            ano: "",
        },
        defaultItem: {
            nome: "",
            marca_id: "",
            modelo: "",
            ano: "",
        },
        carros: [],
        marcas: [],
        valid: true,
        nameRules: [
            (v) => !!v || "Nome é obrigatório",
            (v) =>
                (v && v.length <= 30) ||
                "O nome deve possuir no máximo 30 caracteres",
            (v) =>
                (v && v.length >= 3) ||
                "O nome deve possuir no mínimo 3 caracteres",
        ],
        modeloRules: [
            (v) => !!v || "Modelo é obrigatório",
            (v) =>
                (v && v.length == 4) ||
                "Deve ser informado o ano do modelo com exatamente 4 dígitos",
        ],

        anoRules: [
            (v) => !!v || "Ano é obrigatório",
            (v) =>
                (v && v.length == 4) ||
                "Deve ser informado o ano com exatamente 4 dígitos",
        ],
        message: "",
        messageOverlay: "",
        overlay: false,
        dialog1: false,
    }),

    computed: {
        formTitle() {
            return this.editedIndex === -1 ? "Novo item" : "Editar item";
        },
    },

    watch: {
        dialog(val) {
            val || this.close();
        },
        dialogDelete(val) {
            val || this.closeDelete();
        },
    },

    created() {
        this.inicialize();
    },

    methods: {
        inicialize() {
            axios("https://app.desdiogo.com/api/carros").then((res) => {
                this.carros = res.data;
            });
            axios("https://app.desdiogo.com/api/marcas").then((res) => {
                this.marcas = res.data;
            });
        },
        editItem(item) {
            this.editedItem = Object.assign({}, item);
            this.editedIndex = this.editedItem.id;
            this.dialog = true;
        },

        deleteItem(item) {
            this.editedItem = Object.assign({}, item);
            this.editedIndex = this.editedItem.id;
            this.dialogDelete = true;
        },

        deleteItemConfirm() {
            this.messageOverlay = "Deletando o registro";
            this.overlay = true;
            axios
                .delete(`https://app.desdiogo.com/api/carros/${this.editedIndex}`)
                .then((response) => {
                    if (
                        response.request.readyState === 4 &&
                        response.request.status === 200
                    )
                        this.inicialize();
                    this.overlay = false;
                    this.message = "O registro foi deletado com sucesso";
                    this.dialog1 = true;
                });
            this.closeDelete();
        },

        close() {
            this.dialog = false;
            this.$nextTick(() => {
                this.editedItem = Object.assign({}, this.defaultItem);
                this.editedIndex = -1;
            });
            this.reset();
        },

        closeDelete() {
            this.dialogDelete = false;
            this.$nextTick(() => {
                this.editedItem = Object.assign({}, this.defaultItem);
                this.editedIndex = -1;
            });
        },
        validate() {
            {
                if (!this.$refs.form.validate()) {
                    this.$refs.form.validate();
                } else {
                    this.overlay = true;
                    this.save();
                    this.close();
                }
            }
        },
        save() {
            const requestItems = {
                nome: this.editedItem.nome,
                marca_id: this.editedItem.marca_id.id,
                modelo: this.editedItem.modelo,
                ano: this.editedItem.ano,
            };
            if (this.editedIndex === -1) {
                this.messageOverlay = "Salvando Registro";
                axios
                    .post("https://app.desdiogo.com/api/carros", requestItems)
                    .then((response) => {
                        if (
                            response.request.readyState === 4 &&
                            response.request.status === 201
                        )
                            this.inicialize();
                        this.overlay = false;
                        this.message = "O registro foi salvo com sucesso";
                        this.dialog1 = true;
                    });
            } else {
                this.messageOverlay = "Atualizando Registro";
                axios
                    .put(
                        `https://app.desdiogo.com/api/carros/${this.editedIndex}`,
                        requestItems
                    )
                    .then((response) => {
                        if (
                            response.request.readyState === 4 &&
                            response.request.status === 201
                        )
                            this.inicialize();
                        this.overlay = false;
                        this.message = "O registro foi atualizado com sucesso";
                        this.dialog1 = true;
                    });
            }
        },
        reset() {
            this.$refs.form.reset();
        },
    },
};
</script>

<style></style>

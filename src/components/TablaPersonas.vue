<template>
    <v-container>
        <v-data-table :headers="headers" :items="listaData" class="elevation-1">
            <template v-slot:top>
                <v-toolbar flat>
                    <v-toolbar-title>Personas</v-toolbar-title>
                    <v-divider class="mx-4" inset vertical></v-divider>
                    <v-spacer></v-spacer>
                    <v-dialog v-model="dialog" max-width="500px">
                        <template v-slot:activator="{ props }">
                            <v-btn color="primary" dark class="mb-2" v-bind="props">
                                New Item
                            </v-btn>
                        </template>
                        <v-card>
                            <v-card-title>
                                <span class="text-h5">{{ formTitle }}</span>
                            </v-card-title>
    
                            <v-card-text>
                                <v-container>
                                    <v-row>
                                        <v-col cols="12" sm="6" md="4">
                                            <v-text-field v-model="editedItem.nombre" label="Nombre"></v-text-field>
                                        </v-col>
                                        <v-col cols="12" sm="6" md="4">
                                            <v-text-field v-model="editedItem.imagen" label="Imagen"></v-text-field>
                                        </v-col>
                                        <v-col cols="12" sm="6" md="4">
                                            <v-text-field v-model="editedItem.rfc" label="RFC"></v-text-field>
                                        </v-col>
                                    </v-row>
                                </v-container>
                            </v-card-text>
    
                            <v-card-actions>
                                <v-spacer></v-spacer>
                                <v-btn color="blue-darken-1" variant="text" @click="close">
                                    Cancel
                                </v-btn>
                                <v-btn color="blue-darken-1" variant="text" @click="save">
                                    Save
                                </v-btn>
                            </v-card-actions>
                        </v-card>
                    </v-dialog>
                    <v-dialog v-model="dialogDelete" max-width="500px">
                        <v-card>
                            <v-card-title class="text-h5">Are you sure you want to delete this item?</v-card-title>
                            <v-card-actions>
                                <v-spacer></v-spacer>
                                <v-btn color="blue-darken-1" variant="text" @click="closeDelete()">Cancel</v-btn>
                                <v-btn color="blue-darken-1" variant="text" @click="deleteItemConfirm()">OK</v-btn>
                                <v-spacer></v-spacer>
                            </v-card-actions>
                        </v-card>
                    </v-dialog>
                </v-toolbar>
            </template>
            <template v-slot:item.actions="{ item }">
                <v-icon size="small" class="me-2" @click="editItem(item.raw)">
                    mdi-pencil
                </v-icon>
                <v-icon size="small" @click="deleteItem(item.raw)">
                    mdi-delete
                </v-icon>
            </template>
            <template v-slot:no-data>
                <v-btn color="primary" @click="reset">
                    Reset
                </v-btn>
            </template>
        </v-data-table>
    </v-container>
</template>






<script setup>
    /*
    * Importaciones agregadas:
    * ref: necesario para lograr inteconectar variables
    * con la Composition API.
    * watch: Permite llamar una funcion cuando algun elemento
    * reactivo cambia.
    * computed: Permite decidir con una funcion, como actuar
    * con elementos reactivos para mostrar informacion
    * condicional a estos.
    * onMounted: Permite un despliege inicial seguro, con
    * la informacion de la api.
    */ 
    import { ref,watch,computed,onMounted } from 'vue';
    let posteos;
    let usuarios;
    const listaNombres = ref(null);
    const listaPost = ref(null);
    const listaData = ref([]);
    const dialog = ref(false);
    const dialogDelete = ref(false);
    const headers = ref([
        {
            title: 'Nombre',
            align: 'start',
            sortable: false,
            key: 'nombre',
        },
        {
            title: 'Imagen',
            key: 'imagen'
        },
        {
            title: 'RFC',
            key: 'rfc'
        },
        {
            title: 'Actions',
            key: 'actions',
            sortable: false
        },
    ]);
    const editedIndex = ref(-1);
    const editedItem = ref({
        id: 0,
        nombre: "",
        titulo: "",
        post: "",
    });
    const defaultItem = ref({
        id: 0,
        nombre: "",
        titulo: "",
        post: "",
    });

    const formTitle = computed(() => {
        return editedIndex.value === -1 ? 'New Item' : 'Edit Item'
    })
    
    //Funciones basicas para la tabla 
    function editItem(item) {
        editedIndex.value = listaData.value.indexOf(item);
        editedItem.value = Object.assign({}, item);
        dialog.value = true;
    }

    function deleteItem(item) {
        editedIndex.value = listaData.value.indexOf(item);
        editedItem.value = Object.assign({}, item);
        dialogDelete.value = true;
    }

    /* 
    * Por como funciona, es facil conseguir el ID
    * para pedir la solicitud de borrado al servidor.
    */ 
    async function deleteItemConfirm() {
        console.log(listaData.value[editedIndex.value].id);
        await fetch("https://jsonplaceholder.typicode.com/posts/" + listaData.value[editedIndex.value].id,
        {method: 'DELETE',}).catch(error => alert(error));
        listaData.value.splice(editedIndex.value, 1);
        posteos.splice(editedIndex.value, 1);
        closeDelete();
    }

    function close() {
        dialog.value = false;
        editedItem.value = Object.assign({}, defaultItem.value);
        editedIndex.value = -1;
    }

    function closeDelete() {
        dialogDelete.value = false;
        editedItem.value = Object.assign({}, defaultItem.value);
        editedIndex.value = -1;
    }

    // Identifica si el autor ya esta registrado.
    function getIdPorNombre(nombreBuscado){
        let i = 0;
        let encontrado = false;
        let id = -1;
        do{
            if(usuarios[i].name == nombreBuscado){
                encontrado = true;
                id = usuarios[i].id;
            }
            i++;
        }while((i < usuarios.length) && (!encontrado));
        return id;
    }

    /* 
    * Esta funcion esta acondicionada para que se pueda aprovechar
    * tanto para nuevos registros, como simples ediciones, en ambos
    * casos se implementa un sistema de indentificacion para saber
    * si se nececita realizar un fetch extra para registrar un autor
    * mas a la base de datos.
    */ 
    async function save() {
        if (editedIndex.value > -1) { //Es una edicion de un registro
            let idNombre = getIdPorNombre(editedItem.value.name);
            if (idNombre === -1){
                fetch('https://jsonplaceholder.typicode.com/users', {
                    method: 'POST',
                    body: JSON.stringify({
                    id: usuarios.length+1,
                    name: editedItem.value.name,
                }),
                headers: {
                    'Content-type': 'application/json; charset=UTF-8',
                },
                })
                .then((response) => response.json())
                .then((json) => console.log(json))
                .catch(error => alert(error));
                
                usuarios.push({id:usuarios.length+1 , name:editedItem.value.name});
            }
            fetch('https://jsonplaceholder.typicode.com/posts', {
                method: 'PUT',
                body: JSON.stringify({
                title: editedItem.value.titulo,
                body: editedItem.value.post,
                userId: idNombre,
            }),
            headers: {
                'Content-type': 'application/json; charset=UTF-8',
            },
            }).then((response) => response.json()).then((json) => console.log(json)).catch(error => alert(error));
            Object.assign(listaData.value[editedIndex.value], editedItem.value);
            Object.assign(posteos[editedIndex.value], editedItem.value);
        } 
        else { // Es un nuevo registro.
            let idNombre = getIdPorNombre(editedItem.value.name);
            if (idNombre === -1){
                fetch('https://jsonplaceholder.typicode.com/users', {
                    method: 'POST',
                    body: JSON.stringify({
                    id: usuarios.length+1,
                    name: editedItem.value.name,
                }),
                headers: {
                    'Content-type': 'application/json; charset=UTF-8',
                },
                }).then((response) => response.json()).then((json) => console.log(json)).catch(error => alert(error));
                usuarios.push({id:usuarios.length+1 , name:editedItem.value.name});
            }
            fetch('https://jsonplaceholder.typicode.com/posts', {
                method: 'POST',
                body: JSON.stringify({
                title: editedItem.value.titulo,
                body: editedItem.value.post,
                userId: idNombre,
            }),
            headers: {
                'Content-type': 'application/json; charset=UTF-8',
            },
            }).then((response) => response.json()).then((json) => console.log(json)).catch(error => alert(error));
            listaData.value.push(editedItem.value);
            posteos.push(editedItem.value);
        }
        close();
    }

    function reset() {
        Object.assign(listaData.value,posteos);
    }

    watch(
        dialog,async(val) => {
            val || close();
        },
        dialogDelete,async(val) => {
            val || closeDelete();
        },
    )

    /* 
    * Funcion que recupera la informacion de la base de datos
    * mediante una promesa.
    */ 
    onMounted(async() => {
        let token = localStorage.getItem('jwtToken');
        console.log(`Bearer ${token}`);
        await fetch("https://localhost:3000/personas",{
            method: 'GET',
            headers: {
                'Authorization': `Bearer ${token}`,
                'Content-Type': 'application/json',
            },
        })
        //.then((response) => response.json()).then((json) => console.log(json));
        // usuarios = JSON.parse(listaNombres.value);
        // posteos = JSON.parse(listaPost.value);
        //Genera un array listo para colocar los datos en la tabla
        // posteos = posteos.map(function(post){
        //     let nombre;
        //     //Cambiare esto algun dia por un do while... 
        //     //o un break (si es que existe en javascript).
        //     usuarios.forEach(usuario => {
        //         if(usuario.id == post.userId)
        //             nombre = usuario.name;
        //     });
        //     return {id:post.id,nombre:nombre,titulo:post.title,post:post.body};
        // })
        // Object.assign(listaData.value,posteos);
    })
</script>

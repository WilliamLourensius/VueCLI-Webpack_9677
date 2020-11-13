<template>
    <v-main class="list">
        <h3 class="text-h3 font-weight-medium mb-5">To Do List</h3>

        <v-card>
            <v-card-title>
                 <v-text-field
                  v-model="search"
                  append-icon="mdi-magnify"
                  label="Search"
                  single-line
                  hide-details
                  >
                 </v-text-field>
                 <v-spacer></v-spacer>
                  <v-select
                    :items="['Penting','Tidak penting']"
                    v-model="sort"
                    label="Urutkan"
                    @change="sortedArray(sort)"
                    outlined
                    hide-details
                    class="mr-3"
                    dense></v-select>
                 <v-btn color="success" dark @click="dialog = true">
                     Tambah
                 </v-btn>
            </v-card-title>    
            <v-data-table :headers="headers" :items="todos" :search="search"
            :single-expand="singleExpand"
                :expanded.sync="expanded"
                item-key="note"
                show-expand
                class="elevation-1">
                <template v-slot:expanded-item="{ item }">
                <h3>Note :</h3>
                {{ item.note }}                        
                </template>
                 <template v-slot:[`item.priority`]="{ item }">        
                    <v-chip outlined  :color="warna(item.priority)">{{item.priority}}</v-chip>               
                </template>
                <template v-slot:[`item.actions`]="{item}">
                    <v-btn icon small class="mr-2" @click="editItem(item)">
                      <v-icon color="indigo">mdi-pencil</v-icon>
                    </v-btn>
                    <v-btn icon small @click="deleteItem(item)">
                         <v-icon color="red">mdi-delete</v-icon>
                    </v-btn>
                </template>
                <template v-slot:[`item.checks`]="{item}">
                    <v-checkbox multiple :key="item" @click.capture.stop="checkItem(item)"/>              
                </template>
            </v-data-table>
        </v-card>
        <v-card v-if="check.length" class="mt-5" style="overflow-y:scroll; height:200px">
        <v-card-title class="float-left">Delete Multipe</v-card-title>
         <v-btn color="error ml-4 mt-5 mr-5 float-right" dark @click="deleteAll">
                Hapus Semua
            </v-btn>
            <br>
            <br>
            <br>
            <v-list-item v-for="(item, i) in check" :key="i">
                    <v-list-item-content>
                        <v-list-item-title> - {{item.task}}</v-list-item-title>
                    </v-list-item-content>
            </v-list-item>
        </v-card>
        <v-dialog v-model="dialogDel" persistent max-width="420px">
            <v-card>
                <v-card-title>
                    <span class="headline">Apakah anda yakin menghapus ?</span>
                </v-card-title>                                         
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="green darken-1" text @click="cancel">
                        No
                    </v-btn>
                    <v-btn color="red darken-1" text @click="yes">
                        Yes
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        <v-dialog v-model="dialogEd" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">Form Todo - Update</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-text-field
                        v-model="formTodo.task"
                        label="Task"
                        required>                        
                        </v-text-field>
                        <v-select
                        v-model="formTodo.priority"
                        :items="['Penting','Biasa','Tidak penting']"
                        label="Priority"
                        required>
                        </v-select>
                        <v-textarea 
                        v-model="formTodo.note"
                        label="Note"
                        required>                     
                        </v-textarea>     
                    </v-container>                        
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="cancel">
                        Cancel
                    </v-btn>
                    <v-btn color="blue darken-1" text @click="update">
                        Update
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        <v-dialog v-model="dialog" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="headline">Form Todo - Add</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-text-field
                        v-model="formTodo.task"
                        label="Task"
                        required>                        
                        </v-text-field>
                        <v-select
                        v-model="formTodo.priority"
                        :items="['Penting','Biasa','Tidak penting']"
                        label="Priority"
                        required>
                        </v-select>
                        <v-textarea 
                        v-model="formTodo.note"
                        label="Note"
                        required>                     
                        </v-textarea>     
                    </v-container>                        
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="cancel">
                        Cancel
                    </v-btn>
                    <v-btn color="blue darken-1" text @click="save">
                        Save
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </v-main>
</template>
<script>
export default {
    name:"List",
    data(){
        return {
            expanded: [],                  
            search:null,
            dialog: false,
            dialogDel: false,
            dialogEd: false,    
            sort:"unsort",
            singleExpand: false,    
            check:[],   
            cekPoint:null, 
            headers:[
                {
                    text: "Task",
                    align: "start",
                    sortable: true,
                    value: "task",
                },
                { text: "Priority", value:"priority"},                
                { text: "Actions", value: "actions"},
                { text: "", value: "checks"},
            ],
            todos: [
                {                    
                    task: "bernafas",
                    priority: "Penting",
                    note: "huffttt",
                },
                {
                    task: "nongkrong",
                    priority: "Tidak penting",
                    note: "bersama tman2",
                },
                {
                    task: "masak",
                    priority:"Biasa",
                    note:"masak air 500ml",
                },
            ],
            formTodo: {
                task: null,
                priority:null,
                note:null,
            },
            simpan : [],
        };
    },
    methods:{
        deleteItem(item){
            this.dialogDel = true;
            this.simpan = item;

        },
        warna:function(item){
            if(item == "Penting")
                return "red";
            else if(item =="Biasa")
                return "blue";
            else if(item == "Tidak penting")
                return "green";
        },
        yes: function() {
             this.todos.splice(this.todos.indexOf(this.simpan), 1);
             this.dialogDel = false;
        },
        editItem(item){
            this.formTodo.task = item.task;
            this.formTodo.priority = item.priority;
            this.formTodo.note = item.note;
            this.dialogEd = true;
            this.simpan = item;
        },
        update(){
            this.simpan.task=this.formTodo.task;
            this.simpan.priority= this.formTodo.priority;
            this.simpan.note = this.formTodo.note;
            this.dialogEd = false;
        },
        save() {
            this.todos.push(this.formTodo);
            this.resetForm();
            this.dialog = false;
        },
        cancel(){
            this.resetForm();
            this.dialog = false;
            this.dialogDel = false;
            this.dialogEd = false;
        },
        resetForm() {
            this.formTodo = {
                task: null,
                priority: null,
                note: null,
            };
        },
        checkItem(item){
            if(this.check.includes(item)) {
                this.check.splice(this.check.indexOf(item), 1);
            } else {
                this.check.push(item);                
            }
        },
        deleteAll:function(){             
            for(var i = 0; i < this.check.length; i++){            
                this.todos.splice( this.todos.indexOf(this.check[i]), 1);
            }
            this.check=[];                            
        },
        sortedArray: function(sort) {                                  
                function compare(a,b) {                          
                      if(sort == "Tidak penting"){           
                        if (a.priority == "Tidak penting")
                            return -1;                     
                        if (a.priority == "Penting")
                            return 1;                                           
                        if (a.priority == "Biasa" && b.priority=="Tidak penting")
                            return 1;                       
                        if (a.priority == "Biasa" && b.priority=="Penting")
                            return 0;                                                                                                    
                      }
                      if(sort == "Penting"){
                        if (a.priority == "Penting")
                            return -1;
                        if (a.priority == "Tidak penting")
                            return 1;                                       
                        if (a.priority == "Biasa" && b.priority=="Penting")
                            return 1;                       
                        if (a.priority == "Biasa" && b.priority=="Tidak penting")
                            return 0;
                      }                                            
                }
                return this.todos.sort(compare);                                                       
        }
    }
};
</script>
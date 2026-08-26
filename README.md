# nexo-foundation-workspace
/* ==========================================================
   NEXO Foundation Workspace
   storage.js
   Camada de persistência local
   ========================================================== */

const STORAGE_KEY = "nexo.foundation.database";


const DEFAULT_DATABASE = {
    version: "1.0.0-dev",

    documents: [],

    settings: {
        theme: "dark",
        lastDocument: null
    }
};


/* ==========================================================
   Utilidades
   ========================================================== */

function cloneObject(obj){

    return JSON.parse(
        JSON.stringify(obj)
    );

}


function generateId(){

    return (

        Date.now().toString(36)

        +

        Math.random()
        .toString(36)
        .substring(2)

    );

}


/* ==========================================================
   Banco
   ========================================================== */

let database = loadDatabase();


function loadDatabase(){

    const saved =
        localStorage.getItem(
            STORAGE_KEY
        );


    if(!saved){

        const fresh =
            cloneObject(
                DEFAULT_DATABASE
            );

        localStorage.setItem(
            STORAGE_KEY,
            JSON.stringify(fresh)
        );

        return fresh;

    }


    try{

        return JSON.parse(saved);

    }

    catch(error){

        console.error(
            "Erro carregando banco:",
            error
        );


        return cloneObject(
            DEFAULT_DATABASE
        );

    }

}



function saveDatabase(){

    localStorage.setItem(

        STORAGE_KEY,

        JSON.stringify(database)

    );

}


/* ==========================================================
   Documentos
   ========================================================== */

function createDocument(module="dashboard"){

    const document = {

        id: generateId(),

        module,

        title:
            "Novo Documento",

        content:"",

        favorite:false,

        createdAt:
            new Date()
            .toISOString(),

        updatedAt:
            new Date()
            .toISOString()

    };


    database.documents.push(
        document
    );


    database.settings.lastDocument =
        document.id;


    saveDatabase();


    return document;

}



function getDocument(id){

    return database.documents.find(

        document =>
            document.id === id

    );

}



function getDocuments(module=null){

    if(!module){

        return database.documents;

    }


    return database.documents.filter(

        document =>
            document.module === module

    );

}



function updateDocument(id, data){

    const document =
        getDocument(id);


    if(!document){

        return null;

    }


    Object.assign(
        document,
        data
    );


    document.updatedAt =
        new Date()
        .toISOString();


    saveDatabase();


    return document;

}



function deleteDocument(id){

    database.documents =
        database.documents.filter(

            document =>
                document.id !== id

        );


    if(
        database.settings.lastDocument === id
    ){

        database.settings.lastDocument =
            null;

    }


    saveDatabase();

}


/* ==========================================================
   Pesquisa
   ========================================================== */

function searchDocuments(query){

    if(!query){

        return database.documents;

    }


    query =
        query.toLowerCase();



    return database.documents.filter(

        document =>

            document.title
            .toLowerCase()
            .includes(query)

            ||

            document.content
            .toLowerCase()
            .includes(query)

    );

}


/* ==========================================================
   Inicialização
   ========================================================== */

if(
    database.documents.length === 0
){

    createDocument(
        "dashboard"
    );

}

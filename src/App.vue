<template>
    <div class=curdir>{{directory}}</div>
    <div style="padding: 5vw 10vh;">
        <div v-if="directory != `Выберете хранилище данных.`" class="list-group-item list-group-item-action shadow dirents" v-on:click="(event) => changeDir(event)">
           <div class="dirent">
               <i class="bi bi-arrow-90deg-up"></i> <span id="folder_name">..</span>
           </div>
        </div>
        <div v-for="folder in directoryContents[0]" v-bind:key="folder.id" v-on:click="(event) => changeDir(event)" class="list-group-item list-group-item-action shadow dirents" >
            <div class="dirent">
                <i class="bi bi-folder"></i> <span id="folder_name">{{folder}}</span>
            </div>
        </div>
        <div v-for="file in directoryContents[1]" v-bind:key="file.id" class="list-group-item list-group-item-action shadow dirents" style="display: flex; justify-content: space-between">
            <div>
                <i class="bi" :class="file.icon"></i> {{file.name}}
            </div>
            <span>{{file.size}}</span>
        </div>
    </div>
</template>

<script>
const fs = require("fs");
const { join } = require("path");
const mime = require("mime-types");

const formatSize = (size) => {
    var i = Math.floor(Math.log(size) / Math.log(1024));
    return (size / Math.pow(1024, i)).toFixed(2) * 1 + " " + ["B", "kB", "MB", "GB", "TB"][i];
};
const getFilesandDirs = (path) => {
    console.log(`getFilesandDirs was ran with path ${path}`)
    const dirs = fs
        .readdirSync(path, { withFileTypes: true })
        .filter((item) => item.isDirectory())
        .map((item) => item.name);
    
    const getIcon = (filename) => {
        const mimetype = mime.lookup(filename);
        
        const icon_classes = {
            "default": "bi-file-earmark",
            "image": "bi-file-earmark-image",
            "audio": "bi-file-earmark-music",
            "video": "bi-file-earmark-play",
            "text": "bi-file-earmark-text",
            "application/pdf": "bi-file-earmark-pdf"
        }

        if(!mimetype) return icon_classes.default;
        
        return icon_classes[mimetype.split("/")[0]] || icon_classes[mimetype] || icon_classes.default
    }

    const files = fs
        .readdirSync(path, { withFileTypes: true })
        .filter((item) => item.isFile())
        .map((item) => {
            const icon = getIcon(item.name);
            let size, sizeFormatted;
            try {
                size = fs.statSync(join(path, item.name)).size;
                if(size != 0) sizeFormatted = formatSize(size);
                else sizeFormatted = "0 B";
            } catch(err) {
                sizeFormatted = "?";
            }

            return { name: item.name, icon: icon, size: sizeFormatted };
        });
    return [dirs, files];
};


export default {
    data() {
        let directory = "Выберете хранилище данных.";
        let directoryContents = [["C:\\", "D:\\"], []];
        return {
            directory, directoryContents
        };
    },
    methods: {
        changeDir(message, event) {
            if (event) event.preventDefault();

            console.log(message.target.id);

            let targetDirectoryName;

                if(message.target.classList.contains("dirents"))
                    targetDirectoryName = message.target.children[0].children[1].innerText;
                else if(message.target.classList.contains("dirent"))
                    targetDirectoryName = message.target.children[1].innerText;
                else if(message.target.id == "folder_name")
                    targetDirectoryName = message.target.innerText;

            if((this.directory == "C:\\"||this.directory == "D:\\") && targetDirectoryName == "..") {
                this.directory = "Выберете хранилище данных:";
                this.directoryContents = [["C:\\", "D:\\"], []];
                return;
            } else {
                const targetDirectoryPath = require("path").resolve(this.directory, targetDirectoryName);

                this.directoryContents = getFilesandDirs(targetDirectoryPath);
                this.directory = targetDirectoryPath;
            }
        }
    },
    name: "App",
    components: {}
};
</script>

<style>
.curdir
{
    text-align: center;
    font-family: Arial, Helvetica, sans-serif;
    font-size: 3em;
}
.file_field_btn {
    width: 100%;
    background-color: rgba(255, 255, 255, 0);
    border-block-color: rgba(255, 255, 255, 0);
    border-color: rgba(255, 255, 255, 0);
}
.dirents {
    text-align: left !important;
    user-select: none;
    font-family: Arial, Helvetica, sans-serif;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
::-webkit-scrollbar {
    width: 10px;
}

::-webkit-scrollbar-track {
    -webkit-box-shadow: inset 0 0 100px rgba(73, 66, 66, 0.301); 
    border-radius: 1px;
}

::-webkit-scrollbar-thumb {
    border-radius: 1px;
    width: 6px;
    -webkit-box-shadow: inset 0 0 100px rgb(0, 0, 0); 
}

</style>

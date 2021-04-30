<template>
    <div>
        <div class="hidden">
            <input id="input_file" type="file" @change="onFileChange($event)"><br>
        </div>

        <tui-image-editor ref="editor" 
            :include-ui="useDefaultUI" 
            :options="options"></tui-image-editor>


        <div class="hidden">
            <span id="tpl_button">
                <button id="btn__upload" class="tui-image-editor-download-btn" style="background-color: #FFF;border: 1px solid #303F9F;color: #303F9F;font-family: 'Noto Sans', sans-serif;font-size: 12px">
                    Upload
                </button>
                <button id="btn__download" class="tui-image-editor-download-btn" style="background-color: #303F9F;border: 1px solid #303F9F;color: #fff;font-family: 'Noto Sans', sans-serif;font-size: 12px">
                    Download
                </button>
            </span>
        </div>

        <div class="hidden">
            <div id="tpl_textarea" style="text-align:left;padding-left: 30px;">
                <h4>Tatacara Penggunaan</h4>
                <ol>
                    <li>Upload sertifikat</li>
                    <li>Masukkan nama di inputan</li>
                    <li>Pisahkan nama dengan baris</li>
                    <li>Tips: Masukkan nama yang punya jumlah kata sama</li>
                    <li>Atur posisi <b>Nama Peserta</b></li>
                    <li>Tekan tombol download</li>
                </ol>
                <br>
                <textarea cols="30" rows="10" placeholder="Masukkan nama peserta disini, pisahkan dengan baris">
                </textarea>
                {{ nama_peserta }}
            </div>
        </div>
    </div>
</template>

<script>
import {ImageEditor} from '@toast-ui/vue-image-editor';
import JSZip from 'jszip';
import { saveAs } from 'file-saver';

export default {
    name: "EditorTui",
    components: {
        'tui-image-editor': ImageEditor
    },
    data() {
        const whiteTheme = { 'common.bi.image': 'https://uicdn.toast.com/toastui/img/tui-image-editor-bi.png',
            'common.bisize.width': '251px',
            'common.bisize.height': '21px',
            'common.backgroundColor': '#f7f7f7',
            'common.border': '1px solid #c1c1c1',

            // header
            'header.backgroundImage': 'none',
            'header.backgroundColor': 'transparent',
            'header.border': '0px',

            // load button
            'loadButton.backgroundColor': '#fff',
            'loadButton.border': '1px solid #ddd',
            'loadButton.color': '#222',
            'loadButton.fontFamily': '\'Noto Sans\', sans-serif',
            'loadButton.fontSize': '12px',

            // download button
            'downloadButton.backgroundColor': '#fdba3b',
            'downloadButton.border': '1px solid #fdba3b',
            'downloadButton.color': '#fff',
            'downloadButton.fontFamily': '\'Noto Sans\', sans-serif',
            'downloadButton.fontSize': '12px',

            // main icons
            'menu.normalIcon.color': '#8a8a8a',
            'menu.activeIcon.color': '#555555',
            'menu.disabledIcon.color': '#434343',
            'menu.hoverIcon.color': '#e9e9e9',
            'menu.iconSize.width': '24px',
            'menu.iconSize.height': '24px',

            // submenu icons
            'submenu.normalIcon.color': '#8a8a8a',
            'submenu.activeIcon.color': '#555555',
            'submenu.iconSize.width': '32px',
            'submenu.iconSize.height': '32px',

            // submenu primary color
            'submenu.backgroundColor': 'transparent',
            'submenu.partition.color': '#e5e5e5',

            // submenu labels
            'submenu.normalLabel.color': '#858585',
            'submenu.normalLabel.fontWeight': 'normal',
            'submenu.activeLabel.color': '#000',
            'submenu.activeLabel.fontWeight': 'normal',

            // checkbox style
            'checkbox.border': '1px solid #ccc',
            'checkbox.backgroundColor': '#fff',

            // rango style
            'range.pointer.color': '#333',
            'range.bar.color': '#ccc',
            'range.subbar.color': '#606060',

            'range.disabledPointer.color': '#d3d3d3',
            'range.disabledBar.color': 'rgba(85,85,85,0.06)',
            'range.disabledSubbar.color': 'rgba(51,51,51,0.2)',

            'range.value.color': '#000',
            'range.value.fontWeight': 'normal',
            'range.value.fontSize': '11px',
            'range.value.border': '0',
            'range.value.backgroundColor': '#f5f5f5',
            'range.title.color': '#000',
            'range.title.fontWeight': 'lighter',

            // colorpicker style
            'colorpicker.button.border': '0px',
            'colorpicker.title.color': '#000'
        };

        return {
            nama_peserta: "",
            text_id: null,
            useDefaultUI: true,
            options: { // for tui-image-editor component's "options" prop
                cssMaxWidth: 700,
                cssMaxHeight: 500,
                includeUI: {
                    theme: whiteTheme,
                    menuBarPosition: "left",
                    menu: ['text'],
                    loadImage: {
                        path: "./placeholder.png",
                        name: "placeholder"
                    }
                },
            }
        };
    },
    mounted() {
        this.initUI()
    },
    methods: {
        initUI() {
            document.querySelector('.tui-image-editor-header-logo').innerHTML = "<h1 class='logo__title'>Generator Sertifikat <small>&mdash; v0.1</small></h1>"

            let target = document.querySelector('#tpl_button').cloneNode(true)
            document.querySelector('.tui-image-editor-header-buttons').innerHTML = ""
            document.querySelector('.tui-image-editor-header-buttons').appendChild(target)
            document.querySelector('.tui-image-editor-header-buttons #btn__download').addEventListener('click', this.downloadImage)
            document.querySelector('.tui-image-editor-header-buttons #btn__upload').addEventListener('click', this.btnUploadClicked)
        
            document.querySelector('li[tooltip-content="Undo"]').remove()
            document.querySelector('li[tooltip-content="Redo"]').remove()
            document.querySelector('li[tooltip-content="Reset"]').remove()
            document.querySelector('li[tooltip-content="Delete"]').remove()
            document.querySelector('li[tooltip-content="DeleteAll"]').remove()
            document.querySelectorAll('.tui-image-editor-item').forEach((e) => {
                if(e.getAttribute('tooltip-content') == null) e.remove()
            });

            let textarea = document.querySelector('#tpl_textarea').cloneNode(true)
            document.querySelector('.tui-image-editor-menu-text').appendChild(textarea)
            document.querySelector('.tui-image-editor-menu-text textarea').addEventListener('change', this.updateNamaPeserta)
        },
        addText() {
            let that = this;
            this.$refs.editor.invoke('addText', 'Nama Peserta', {
                styles: {
                    fontSize: 100,
                    textAlign: 'center'
                }
            }).then(objectProps => {
                that.text_id = objectProps.id
            });
        },
        updateNamaPeserta() {
            this.nama_peserta = document.querySelector('.tui-image-editor-menu-text textarea').value
        },
        btnUploadClicked() {
            document.getElementById('input_file').click()
        },
        onFileChange(e) {
            let files = e.target.files || e.dataTransfer.files;
            if (!files.length)
                return;

            var fr = new FileReader();  
            fr.onload = () => {
                this.$refs.editor.invoke('loadImageFromURL', fr.result, 'Gambar');
                setTimeout(() => this.addText(), 1000)
            };
            fr.readAsDataURL(files[0]);
        },
        downloadImage() {
            let list_nama = this.nama_peserta;
            let arr_nama = list_nama.split(/\r?\n/);

            var zip = new JSZip();
            let req = arr_nama.map((nama, index) => {
                return new Promise((resolve) => {
                    setTimeout(() => {
                        this.$refs.editor.invoke('changeText', this.text_id, nama);
                        let url = this.$refs.editor.invoke('toDataURL');
                        zip.file(`${index}-${nama}.png`, url.replace(/^data:image\/(png|jpg);base64,/, ""), {base64: true});
                        
                        console.log('done with', nama);
                        resolve();
                    }, 100);
                });
            });
            Promise.all(req).then(() => {
                zip.generateAsync({type:"blob"})
                    .then(function(content) {
                        saveAs(content, "Sertifikat.zip");
                    });
                this.$refs.editor.invoke('changeText', this.text_id, "Nama Peserta");
            });
        }
    }
}
</script>

<style type="text/css">
.hidden {
    display: none;
}
.logo__title {
    color: #555;
    font-size: 20px;
}
.tui-image-editor-container.left .tui-image-editor-controls li, .tui-image-editor-container.right .tui-image-editor-controls li {
    background: #FFF;
}
.tui-image-editor-container .tui-image-editor-controls {
    background-color: #303F9F !important;
}
.tui-image-editor-container .tui-image-editor-range-wrap label {
    color: #555 !important; 
}
.tui-image-editor-container.left .tui-image-editor-submenu .tui-image-editor-submenu-item li, .tui-image-editor-container.right .tui-image-editor-submenu .tui-image-editor-submenu-item li{
    margin-top: 7px !important;
}
</style>
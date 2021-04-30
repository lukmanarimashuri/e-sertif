<template>
    <div>
        <input type="text" v-model="text" @change="addText">
        {{ text }}

        <br><br>
        <input type="file" @change="onFileChange(image, $event)"><br>
        <button @click="exportToImage()">Export</button>

        <canvas id="image-canvas" ref="imagecanvas">
        </canvas>
    </div>
</template>

<script>
import { fabric } from 'fabric';

export default {
    name: "Editor",
    data() {
        return {
            text: "",
            image: ""
        }
    },
    mounted() {
        this.initCanvas();
        this.handleResize();
        window.addEventListener('resize', this.handleResize);
    },
    computed: {
        canvas() {
            return new fabric.Canvas(this.$refs.imagecanvas,{
                backgroundColor: '#c8c8c8'
            });
        },
        ctx() {
            return this.canvas.getContext('2d');
        }
    },
    methods: {
        initCanvas() {
            // const rect = new fabric.Rect({
            //     fill: 'red',
            //     width: 50,
            //     height: 50
            // });
            // this.canvas.add(rect);
        },
        addText() {
            if(this.canvas.getObjects()[1] != undefined) {
                this.canvas.remove(this.canvas.getObjects()[1])
            }

            const fb_text = new fabric.Text(this.text, {  
                fill: 'black'
            });
            this.canvas.centerObject(fb_text);
            this.canvas.add(fb_text);
        },
        onFileChange(item, e) {
            let files = e.target.files || e.dataTransfer.files;
            if (!files.length)
                return;
            this.createImage(item, files[0]);
        },
        createImage(item, file) {
            let reader = new FileReader();
            let that = this;

            reader.onload = (e) => {
                var imgObj = new Image();
                imgObj.src = e.target.result;
                imgObj.onload = function () {
                    var image = new fabric.Image(imgObj);
                    image.scaleToWidth(800);
                    image.filters.push(new fabric.Image.filters.Resize({
                        resizeType: 'lanczos',
                        scaleX: .9,
                        scaleY: .9
                    }));
                    image.applyFilters();

                    that.canvas.centerObject(image);
                    that.canvas.add(image);
                    that.canvas.renderAll();
                }
            };
            reader.readAsDataURL(file);
        },
        removeImage: function (item) {
            item.image = false; 
        },
        exportToImage() {
            this.canvas.backgroundColor = 'rgba(0,0,0,0)';
            const string = this.canvas.toDataURL({format: 'png' });
            const iframe = "<iframe width='100%' height='100%' src='" + string + "'></iframe>"
            var x = window.open();
            x.document.open();
            x.document.write(iframe);
            x.document.close();
            this.canvas.backgroundColor = '#c8c8c8';
        },
        handleResize() {
            this.canvas.setHeight(window.innerHeight - 30);
            this.canvas.setWidth(window.innerWidth - 30);
        }
    },
    beforeDestroy() {
        window.removeEventListener('resize', this.handleResize);
    }
}
</script>
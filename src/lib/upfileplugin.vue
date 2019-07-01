<template>
    <div class="box" :style="{'backgroundImage': 'url('+bg+')', 'backgroundSize': num}">
        <span v-show="!filePhoto" @click="close" :style="{'backgroundImage': 'url('+closeImg+')'}" class="close"></span>
        <input v-show="filePhoto" type="file" @change="fn($event)" capture="camera" class="photoBtn" accept="image/*">
    </div>
</template>

<script>
import EXIF from 'exif-js';
export default {
    name: 'upfileplugin',
    data () {
        return {
            bg: require('./add.png'),
            closeImg: require('./del.png'),
            filePhoto: true,
            num: 'auto',
            bgSrc:'',
            showAddCar: false
        } 
    },
    components: {
        zoomImgCommon
    },
    methods: {
        close () {
            this.filePhoto = true;
            this.bg = require('./add.png');
            this.num = 'auto';
        },
        fn (e) {
            let obj = e.target;
            let tagetEle = obj.parentNode;
            this.readURL(obj, tagetEle);
        },
        readURL (input, tagetEle) {
            let self = this;
            if (input.files[0]) {
            // FileReader对象允许Web应用程序异步读取存储在用户计算机上的文件（或原始数据缓冲区）的内容
                var reader = new FileReader();
                var canvas = document.createElement('canvas');
                var ctx = canvas.getContext('2d');
                reader.readAsDataURL(input.files[0]);
                reader.onload = function (e) {
                    var img = new Image();
                    img.src = e.target.result;
                    let deg = {
                        "1": '0',
                        "6": '90',
                        "8": '270',
                        "3": '180',
                    }
                    img.onload = function () {
                        canvas.width = '1000';
                        canvas.height = '1000';
                        EXIF.getData(img, function () {
                            var d = EXIF.getTag(this, "Orientation");
                            // 只有手机等设备拍摄才会有此参数
                            if (d && d != 1) {
                                ctx.translate(900, 0);
                                ctx.rotate(`${deg[d]}` * Math.PI / 180)
                                ctx.drawImage(img, 0, -100, 1000, 1000);
                            } else {
                                ctx.drawImage(img, 0, 0, 1000, 1000);
                            }
                            let data = canvas.toDataURL('image/jpeg');
                            // data:image/jpeg;base64,/9j/4AAQSkZJRgABAQ...9oADAMBAAIRAxEAPwD/AD/6AP/Z"
                        });
                    }
                    self.bg = e.target.result;
                    self.filePhoto = false;
                    input.value = '';
                    self.num = '100% 100%';
                }
            }
        }       
    }
}
</script>
<style lang="stylus" scoped>
.box
    height 120px;
    border 1px solid #ccc;
    background no-repeat center;
    display flex;
    justify-content space-around;
    align-items center;
    position relative
    .photoBtn
        width 70px;
        opacity 0;
    .close
        position absolute;
        width 20px;
        height 20px;
        background no-repeat;
        background-size contain;
        right 0;
        top 0;
    .modelBox
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        position: fixed;
        z-index: 2003;
        background #ccc;
        .modelBox-1
            height 100%
            -webkit-user-select: none;
            -moz-user-select: none;
            -ms-user-select: none;
            user-select: none;
            .imgBg
                position absolute;
                left 0;
                right 0;
                top 0;
                bottom 0;
                max-width 100%;
                max-height 100%;
                margin auto;
</style>

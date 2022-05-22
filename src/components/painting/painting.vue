<style src="./painting.scss"></style>
<template>
        <figure class="painting">
            <img :src="inner.image.src" />
        </figure>
</template>

<script>
import _ from 'lodash';


export default {
    props: ['src', 'effects', 'output'],
    data() {
        return {
            inner: {
                image: document.createElement("img"),
                canvas: document.createElement("canvas"),
                context: null,
                width: 0,
                height: 0,
            },
            original: {
                image: document.createElement("img"),
                canvas: document.createElement("canvas"),
                context: null,
                width: 0,
                height: 0,
            },
            updatingInnerImage: false,
            ratio: 1,
            ratioH: 1,
            ratioV: 1,
        }
    },
    methods: {
        update() {
            if (!this.src) return;

            this.inner.width = this.$el.clientWidth;
            this.inner.height = this.$el.clientHeight;

            this.original.image = document.createElement("img");
            this.original.image.src = this.src;

            this.original.image.onload = (e) => {
                this.original.width = this.original.image.width;
                this.original.height = this.original.image.height;

                if (this.original.width > this.original.height) {
                    this.ratio  = this.inner.width / this.original.width;
                } else {
                    this.ratio = this.inner.height / this.original.height;
                }

                this.ratioH = this.inner.width / this.original.width;
                this.ratioV = this.inner.height / this.original.height;


                this.createInnerImage().then(() => {
                    this.updateInnerImage().then(() => {
                    })
                })
            }
        },
        createInnerImage() {
            this.updatingInnerImage = true;
            return new Promise((resolve, reject) => {
                this.inner.image = document.createElement("img");
                this.inner.canvas = document.createElement("canvas");
                this.inner.canvas.width = this.inner.width;
                this.inner.canvas.height = this.inner.height;

                if (this.original.width > this.original.height) {
                    this.inner.canvas.height = this.original.height*this.ratio;
                } else {
                    this.inner.canvas.width = this.original.width*this.ratio;
                }

                this.inner.context = this.inner.canvas.getContext('2d');

                this.inner.context.save();
                this.inner.context.scale(this.ratio, this.ratio);
                this.inner.context.drawImage(this.original.image, 0,0);
                this.inner.context.restore();

                this.inner.image.src = this.inner.canvas.toDataURL('image/png');
                this.updatingInnerImage = false;

                this.$forceUpdate();
                return resolve(this.inner);
            });
        },
        updateInnerImage() {
            return new Promise((resolve, reject) => {
                if (!this.inner.image.src) {
                    return reject();
                }

                _.each(this.effects, effect => {
                    this.inner.context = effect.fn(this.inner.canvas, _.merge( effect.options, {ratio: this.ratio}))
                    this.inner.image.src = this.inner.canvas.toDataURL('image/png');
                });
                this.$forceUpdate();
                return resolve();
            });
        }
    },
    watch: {
        src: {
            handler(val, oldVal) {
                this.update();
            }
        },
        effects: {
            handler(val, oldVal) {
                this.update();
                // console.log("effects",this.updatingInnerImage);
                //
                // this.updateInnerImage();
            },
            deep: true
        },
    },
    mounted() {
        this.update();
    }
}
</script>
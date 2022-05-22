<style src="./painting.scss"></style>
<template>
    <div class="painting-route">
        <header class="title">
            <h1>P41nt1ng</h1>
        </header>

        <hr>
        <section class="viewport">
            <figure class="image-preview" ratio="1x1">
                <div class="loader-container" :class="[updating ? '__isLoading' : '']">
                    Updating...
                </div>

                <painting :src="image" :effects="effects" :output="result"/>

            </figure>

        </section>

        <aside class="sidebar">
            <div class="options">
                <article class="option-group" name="Input">
                    <div class="group">
                        <div class="option">
                            <label for="imageFile">
                                Upload image
                            </label>
                            <input type="file" name="imageFile" id="imageFile" v-on:change="uploadFile"  v-on:click="triggerResizeEvent">
                        </div>


                        <div class="option">
                            <label for="optionName">
                                Add Effect
                            </label>
                            <select v-model="newEffect">
                                <option :value="effect.name" v-for="(effect, index) in possibleEffects">{{effect.name}}</option>
                            </select>
                        </div>


                        <div class="option">
                            <button type="button" class="button" name="button" v-on:click="addEffect(newEffect)"><span> Effect toevoegen </span></button>
                        </div>
                    </div>
                </article>


                <article class="option-group" name="Effects" v-if="effects.length >= 1">

                    <draggable
                        @start="drag=true"
                        @end="drag=false"
                        item-key="id"
                        v-model="effects"
                    >
                        <template #item="{element,index}">

                            <div class="group __isDraggable" :class="[element.open ? '__isOpen' : '', hasOptions(element) ? '__isCollapsable' : '']" >

                                <strong @click="element.open = !element.open" class="option-title">{{element.name}}
                                    <span class="delete-row" @click="removeEffect(index)">Remove</span>
                                </strong>
                                <hr v-if="index != effects.length-1">

                                <div class="option" v-for="option in element.input" :class="[option.inputType == 'hidden' ? '__isHidden' : '']">

                                <span v-if="option.inputType != 'checkbox'">
                                    <label :for="option.label">
                                    {{option.label}}
                                    </label>
                                    <input :type="option.inputType" :id="option.label" v-model.lazy="element.options[option.label]">
                                </span>
                                <span v-if="option.inputType == 'checkbox'">
                                    <input :type="option.inputType" :id="option.label" v-model.lazy="element.options[option.label]">
                                    <label :for="option.label">
                                        {{option.label}}
                                    </label>
                                </span>
                                </div>

                            </div>

                        </template>
                    </draggable>

                </article>

            </div>
        </aside>
    </div>
</template>



<script>

import painting from './../../components/painting/painting.vue'
import _ from 'lodash';
import { saveAs } from 'file-saver';
import Rainbow  from "rainbowvis.js";
import hexRgb from 'hex-rgb';
import rgbHex from 'rgb-hex';

import draggable from 'vuedraggable'


const getPixelValue = (imageData, x,y) => {

    //Color at (x,y) position
    var color = [];
    color.push(imageData.data[((y*(imageData.width*4)) + (x*4)) + 0]);
    color.push(imageData.data[((y*(imageData.width*4)) + (x*4)) + 1]);
    color.push(imageData.data[((y*(imageData.width*4)) + (x*4)) + 2]);
    // color.push(imageData.data[((y*(imageData.width*4)) + (x*4)) + 3]);


    // color['red'] = imageData.data[((y*(imageData.width*4)) + (x*4)) + 0];
    // color['green'] = imageData.data[((y*(imageData.width*4)) + (x*4)) + 1];
    // color['blue'] = imageData.data[((y*(imageData.width*4)) + (x*4)) + 2];
    // color['alpha'] = imageData.data[((y*(imageData.width*4)) + (x*4)) + 3];

    return color;
}


const greyscale = (pixelData) => {
    return parseInt((pixelData[0] + pixelData[1] + pixelData[2]) / 3);
}


export default {
    components: {painting, draggable},
    data() {
        return {
            image: '',
            updating: false,
            result: {},
            drag:false,
            newEffect: "Polkadots",
            effects: []
        }
    },
    methods: {
        triggerResizeEvent() {
            window.dispatchEvent(new Event("resize"));
        },
        uploadFile(input) {
            if (input && input.target &&  input.target.files &&  input.target.files[0]) {
                var reader = new FileReader();
                this.updating = true;
                reader.onload = (e) => {
                    this.image = e.target.result;
                    this.updating = false;
                }

                reader.readAsDataURL(input.target.files[0]);
            }
        },
        hasOptions(effect) {
            var res = 0;

            effect.input.forEach(input => {
                if (res > 0){
                    return;
                }
                if (input.inputType != 'hidden') {
                    res+=1
                }
            })

            return res > 0;
        },
        removeEffect(index) {
            this.effects.splice(index, 1);
        },
        addEffect(effectName) {
            var effect = _.filter(this.possibleEffects, {name: effectName})[0];
            var options = {}
            _.each(effect.possibleOptions, (option,optionName) => {
                options[optionName] = _.merge( option.default)
            });

            this.effects.push({
                name: effect.name,
                options: options,
                input: _.map(effect.possibleOptions, (option,optionName) => {
                    option.type = option.type;
                    var inputType =  option.type;
                    switch (option.type) {
                        case 'string':
                            inputType = 'text'
                        break;
                        case 'boolean':
                            inputType = 'checkbox'
                        break;
                        case 'gradient':
                            inputType = 'hidden'
                        break;
                    }
                    return {
                        label: optionName,
                        inputType: inputType
                    }
                }),
                fn: effect.fn,

            })
        }
    },
    computed: {
        possibleEffects() {
            return [{
                name: "Dot",
                possibleOptions: {
                    radius: {
                        type: "number",
                        default: 48,
                    },
                    color: {
                        type: "string",
                        default: "#ffffff",
                    },
                    x: {
                        type: "number",
                        default: 0,
                    },
                    y: {
                        type: "number",
                        default: 0,
                    }
                },
                fn: (canvas, options) => {
                    var context = canvas.getContext('2d');
                    var centerX = options.x;
                    var centerY = options.y;
                    context.beginPath();
                    context.arc(centerX, centerY, options.radius * options.ratio, 0, 2 * Math.PI, false);
                    context.fillStyle = options.color;
                    context.fill();
                }
            },{
                name: "Polkadots",
                possibleOptions: {
                    amount: {
                        type: "number",
                        default: 16,
                    },
                    offset: {
                        type: "number",
                        default: 2,
                    },
                    color: {
                        type: "string",
                        default: "#fff",
                    },
                    backgroundColor: {
                        type: "string",
                        default: "",
                    },
                    horizontalOffset: {
                        type: "boolean",
                        default: true,
                    },
                },
                fn: (canvas, options) => {
                    var context = canvas.getContext('2d');



                    var amount = options.amount + options.offset
                    if (amount <= 0) {
                        return;
                    }
                    var imageData = context.getImageData(0, 0, canvas.width, canvas.height);

                    if (options.backgroundColor.length > 0) {
                        context.fillStyle = options.backgroundColor;
                        context.rect(0, 0, canvas.width, canvas.height);
                        context.fill();
                    }


                    var y = amount/2;
                    var x = 0;
                    var yIndex = 0;


                    context.fillStyle = options.color;
                    while (y < canvas.height) {
                        if (options.horizontalOffset && yIndex%2 === 1) {
                            x = 0;
                        } else {
                            x = amount/2;
                        }
                        while (x < canvas.width) {


                            var data = getPixelValue(imageData,Math.round(x),Math.round(y))
                            var size = greyscale(data)/255

                            context.fillStyle = rgbHex(data[0],data[1],data[2]);
                            context.beginPath();
                            context.arc(x, y, size*options.amount, 0, 2 * Math.PI, false);
                            context.fill();

                            x += amount;
                        }
                        y += amount;
                        yIndex++;
                    }
                }
            }, {
                name: "Red blue",
                possibleOptions: {
                    gradient: {
                        type: "hidden",
                        default: ["#ff0000", "#0000ff"],
                    },
                },
                fn: (canvas, options) => {
                    var rainbow =  new Rainbow();


                    rainbow.setNumberRange(0, 255);
                    rainbow.setSpectrum(...options.gradient);

                    var context = canvas.getContext('2d');

                    let imgData = context.getImageData(0, 0, canvas.width, canvas.height);

                    let pixels = imgData.data;

                    function defaultGS(pixels, i) {
                        return parseInt((pixels[i] + pixels[i + 1] + pixels[i + 2]) / 3);
                    }

                    var fn = defaultGS

                    for (var i = 0; i < pixels.length; i += 4) {
                        let lightness = fn(pixels, i);
                        var c = hexRgb(rainbow.colourAt(lightness));
                        pixels[i]     = c.red
                        pixels[i + 1] = c.green
                        pixels[i + 2] = c.blue
                        // pixels[i + 3] = c.alpha
                    }
                    context.putImageData(imgData, 0, 0);

                }

            }]
        }
    },
    mounted() {
        this.addEffect("Polkadots")
    }
}
</script>


<template>
  <div class="customizations-parent-container" ref='parent'>
      <div class="customizations-child-container flex">
        <span id="my-badge" class="p-2 ml-2 mr-2 border badge ">{{tag}}</span>           
        <button @click="backgrounds_dropdown($event, backgrounds)"><span>{{backgrounds.active}}</span> &nbsp; &nbsp; &#9662;</button>
        <button class="btn btn-light border border-right-0 border-top-0 border-bottom-0 no-radius" @click="wrap_in_container()"><span class="icon-insert-template"></span></button>
        <button @click="delete_component()"><span class="icon-delete"></span></button>
        <button @click="text_drop_down($event, components)"><span class="icon-add-solid"></span></button>
        <button @click="close_customizations()"><span class="icon-cross-cancel"></span></button>
    </div>
  </div>
</template>

<script>
import { EventBus } from '@/globals/event-bus.js'
import input_type_text from '../components/inputs/type-text.vue'
import input_type_checkbox from '../components/inputs/type-checkbox.vue'
import input_type_radio from '../components/inputs/type-radio.vue'
import submit_button from '../components/buttons/submit-button.vue'
import text_1 from '../components/texts/text-1.vue'
export default {
    components: {
        input_type_checkbox,
        input_type_text,
        input_type_radio,
        submit_button,
        text_1
    },
    props: {
        e: {
            required: true
        }
    },
    data() {
        return {
            tag: null,
            container: null,
            child_container: null,
            colors: null,
            backgrounds: {
                active: 'Dark Background',
                list: null,
                open: false,
            },
            components: {
                list: ["Input Field", "Checkbox", "Radio Button", "Submit Button", "Label"],
                open: false,
                active: null
            },
        }
    },
    methods: {
        wrap_in_container() {
            let target = $(this.e.target)
            let display = target.css("display")
            let wrapper = $("<div>")
            wrapper.attr("component", "true")
            wrapper.addClass("d-flex container")
            target.wrap(wrapper)
        },
        backgrounds_dropdown(ev, obj) {
            let $this = this
            obj.list = $backgrounds
            let {list_buttons, parent} = this.create_background_list(ev, obj)
            for(let i=0; i<list_buttons.length; i++) {
                list_buttons[i].click(() => {
                    obj.active = $backgrounds[i].name
                    obj.list = $backgrounds[i].list
                    let other_list_buttons = this.create_background_list(ev, obj).list_buttons
                    parent.remove()
                    for(let j=0; j<other_list_buttons.length; j++) {
                        other_list_buttons[j].click(() => {
                            $($this.e.target).css($backgrounds[i].list[j].css)
                        })
                    }
                })
            }
        },
        create_background_list(ev, obj) {
            if(obj.open) return
            let { list } = obj
            list.open = true
            let $this = this
            let parent = $('<div class="customizations-parent-container"></div>')
            let child = $('<div class="customizations-child-container flex align-center"></div>')
            let offset = $(ev.target).offset(),
                height = $(ev.target).outerHeight()
            let list_buttons = []
            for(let i=0; i< list.length; i++) {
                let div = $("<div>")
                div.addClass('customization-background-list')
                div.css(list[i].css)
                child.append(div)
                list_buttons.push(div)
            }
            parent.append(child)
            $(document.body).append(parent)

            parent.css({
                left: offset.left + 'px',
                top: (offset.top + height) + 'px',
                width: "470px",
                maxHeight: "300px",
                overflow: "auto"
            })
            parent.mouseleave(() => {
                parent.remove()
                obj.open = false
            })
            return {
                list_buttons,
                parent
            }
        },
        delete_component() {
            this.close_customizations()
            $(this.e.target).remove()            
        },
        close_customizations() {
            $('.customizations-parent-container').remove()
        },
        text_drop_down(ev, obj) {
            let list_buttons = this.create_simple_list(ev, obj)
            for(let i=0; i<list_buttons.length; i++) {
                list_buttons[i].click(() => {
                    if(obj.list[i] === "Input Field") {
                        EventBus.$emit('$insert_component', { target: this.e.target, container: false, component: input_type_text})
                    }
                    else if(obj.list[i] === "Checkbox") {
                        EventBus.$emit('$insert_component', { target: this.e.target, container: false, component: input_type_checkbox})
                    }
                    else if(obj.list[i] === "Radio Button") {
                        EventBus.$emit('$insert_component', { target: this.e.target, container: false, component: input_type_radio})
                    }
                    else if(obj.list[i] === "Submit Button") {
                        EventBus.$emit('$insert_component', { target: this.e.target, container: false, component: submit_button})
                    }
                    else if(obj.list[i] === "Label") {
                        EventBus.$emit('$insert_component', { target: this.e.target, container: false, component: text_1})
                    }
                })
            }
        },
        create_simple_list(ev, obj) {
            if(obj.open) return
            let { list } = obj
            obj.open = true
            let $this = this
            let parent = $('<div class="customizations-parent-container"></div>')
            let child = $('<div class="customizations-child-container display-block"></div>')
            let offset = $(ev.target).offset(),
                height = $(ev.target).outerHeight()
            let list_buttons = []
            for(let i=0; i< list.length; i++) {
                let button = $("<button>")
                button.html(list[i])
                button.addClass('customization-simple-list')
                child.append(button)
                button.click(() => {
                    obj.active = list[i]
                })
                list_buttons.push(button)
            }
            parent.append(child)
            $(document.body).append(parent)

            parent.css({
                left: offset.left + 'px',
                top: (offset.top + height) + 'px'
            })
            parent.mouseleave(() => {
                parent.remove()
                obj.open = false
            })
            return list_buttons
        },
        
    },
    mounted() {
        this.tag = $(this.e.target).prop('tagName').toLowerCase()
        let $this = this,
            offset = $($this.e.target).offset(),
            height = $($this.e.target).outerHeight()
        this.$nextTick(function() {
            $this.container = $(".customizations-parent-container")
            $this.child_container = $('.customizations-child-container')
            $this.colors = $addColors($this.e.target)
            // $this.child_container.prepend($this.colors)
            $this.colors.insertAfter("#my-badge")
            $this.container.css({
                left: offset.left + 'px',
                top: (offset.top + height) + 'px'
            })

        })
    }
}
</script>

<style>
.customization-background-list {
    width: 150px;
    height: 60px;
    background-color: red;
    margin: 1px;
    display: inline-block;
}
.customization-simple-list {
    width: 100%;
    display: block;
    font-size: 14px;
    text-align: left;
    padding: 5px 20px;
    font-weight: 600;
}
.customization-inline-list {
    font-size: 14px;
    text-align: left;
    padding: 5px;
    font-weight: bold;    
}
.customizations-parent-container {
    position: absolute;
    padding: 10px;
    border: 1px solid rgba(0, 134, 230, 0.111);
    z-index: 2000;
}
.customizations-child-container {
    box-shadow: 0 2px 2px 0 rgba(0,0,0,0.14), 0 3px 1px -2px rgba(0,0,0,0.12), 0 1px 5px 0 rgba(0,0,0,0.2);
    background: #f7f7f7;
    border-radius: 2.5px;
    /* display: flex; */
    flex-wrap: wrap;
    padding: 0px;
    /* padding: 5px; */
}
.customizations-child-container button {
    border: 1px solid transparent;
}
.customizations-child-container button span {
    font-size: 16px;
}
.customizations-child-container button:hover {
    transition: 0.3s;
    border: 1px solid rgba(0, 134, 230, 0.125);
    background: rgba(0, 134, 230, 0.05);
}
.customizations-child-container button:active {
    transition: 0.1s;
    background: rgba(0, 134, 230, 0.125);
}
.customizations-child-container * {
    align-self: center;
}
</style>
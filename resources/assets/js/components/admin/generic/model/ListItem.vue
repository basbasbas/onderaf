<template>
        <!-- Columns: name, type, amount of entries -->
        <!-- Varying column type views: Text / icon, number, tags, images, status, button, options -->
        <tr>
            <td>
                <span class="icon is-small">
                    <i class="fas fa-circle"></i>
                </span>
            </td>
            <td v-for="(prop, key) in model.properties" v-if="!prop.hideInList" v-bind:key="key">
                <component  :is="viewComponent(view(prop, key).type)" 
                            :value="view(prop, key).value"></component>
            </td>
            <td>
                <button class="button is-small is-light">
                    <router-link :to="urls.EDIT(item.id)"> 
                        {{ labels.EDIT }}
                        <!-- <span class="icon is-small">
                            <i class="fas fa-pen"></i>
                        </span> -->
                    </router-link>
                </button>
                <button class="button is-small is-light" @click="remove" v-if="!tempUndeletable">
                    <!-- {{ labels.REMOVE }} -->
                    <span class="icon is-small">
                        <i class="fas fa-times"></i>
                    </span>
                <!-- <a href="javascript:void(0)" @click="remove" v-if="!tempUndeletable">
                </a> -->
                </button>
                <button v-if="modelName == 'entries'" class="button is-small is-light">
                    <router-link :to="urls.CREATE_ENTRY(item.id)"> 
                        <!-- {{ labels.CREATE }} -->
                        <span class="icon is-small">
                            <i class="fas fa-plus"></i>
                        </span>
                    </router-link>
                </button>
                <button v-if="modelName == 'layouts'" class="button is-small is-light">
                    <router-link :to="layoutUrls.CREATE_ENTRY(1)"> 
                        <!-- {{ labels.CREATE }} -->
                        <span class="icon is-small">
                            <i class="fas fa-plus"></i>
                        </span>
                    </router-link>
                </button>
            </td>
            <td>
            </td>
            <td v-if="modelName == 'entries'">
            </td>

        </tr>
</template>

<script>
    import { Models, Texts, ModelRoutingLabelsAndUrls, ModelLabels, ModelEventNames, Components } from "../../../../config.js"

    export default {
        props: ['modelName', 'item'],
        created() {
        },
        computed: {
                model() {
                    return Models[this.modelName] 
                },
                urls() { 
                    return ModelRoutingLabelsAndUrls(this.modelName).urls 
                },
                // temp
                layoutUrls() { 
                    return ModelRoutingLabelsAndUrls('entries').urls 
                },
                labels() { 
                    return ModelLabels(this.modelName) 
                },
                actions() { 
                    return ModelEventNames(this.modelName) 
                },
                // temp
                tempUndeletable() {
                    return this.modelName == 'layouts' && this.item.id == 1
                }
        },
        methods: {
            view(prop, key) {
                // todo make more readable
                let hasType = prop.listView && prop.listView.type,
                    type = hasType ? prop.listView.type : 'text'

                let hasProperty = prop.listView && prop.listView.property
                let value = this.item[key]
                // console.log('vlaue')
                // console.log(key)
                // console.log(this.item)
                

                if (hasProperty) {
                    switch(type) {
                        case 'tags':
                            value = this.item[key].map(x => x[prop.listView.property])
                            break
                        case 'text':
                            if (prop.type == 'relation') {
                                value = this.$store.getters[ModelEventNames(prop.relation.table).FIND](value)[prop.listView.property]
                            } 
                            else {
                                value = this.item[prop.listView.property]
                            }
                            break
                    }
                }

                return {
                    type,
                    value
                }
            },
            viewComponent(type) {
                return Components.view[type]
            },
            remove() {
                this.$store.dispatch(this.actions.REMOVE, this.item.id)
            }
        }
    }
</script>

<style scoped>

</style>
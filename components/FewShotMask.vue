<script setup>
import { mergeProps } from 'vue'
import EmojiPicker from 'vue3-emoji-picker'
import 'vue3-emoji-picker/css'

const menu = ref(false)
const grab = ref(null)
const showEmojiPicker = ref(false)
const props = defineProps({
    maskTitle: {
        type: Array,
        required: true
    },
    fewShotMessages: {
        type: Array,
        required: true
    },
    showButtonGroup: {
        type: Array,
        required: true
    }
})
const emit = defineEmits(['updateAvatar', 'resetTitle'])

const { isMobile } = useDevice()
const pfs = (() => {
    if (isMobile) {
        return {
            l: 'phone-large-font',
            n: 'phone-font',
            s: 'phone-small-font',
            t: 'phone-tiny-font',
            offset: '8 64'
        }
    }
    return { l: '', n: '', s: '', t: '', offset: '16 0' }
})()

const addMessage = () => {
    const fewShotMessage = {
        role: 'user',
        content: ''
    }
    if (props.fewShotMessages.length === 0) {
        fewShotMessage.role = 'system'
        fewShotMessage.content = 'You are a helpful assistant.'
        // showTitle.value = true
    } else if (props.fewShotMessages.length % 2 === 0) {
        fewShotMessage.role = 'assistant'
    }
    props.showButtonGroup.push(true)
    props.fewShotMessages.push(fewShotMessage)
    // Here we use nextTink() before calling scrollIntoView because
    // we need the `fewShotMessages,push` operation reflected in the DOM.
    nextTick(() => {
        grab.value.scrollIntoView({ behavior: 'smooth', block: 'end' })
    })
}

const deleteMessage = (idx) => {
    props.showButtonGroup.splice(idx, 1)
    props.fewShotMessages.splice(idx, 1)
    if (props.fewShotMessages.length === 0) {
        emit('resetTitle')
    }
}

const clearFewShotMask = () => {
    props.showButtonGroup.length = 0
    props.fewShotMessages.length = 0
    emit('resetTitle')
}

const submittingNewMask = ref(false)
const saveFewShotMask = async () => {
    if (props.fewShotMessages.length === 0) {
        return
    }
    submittingNewMask.value = true
    const { data, error } = await useAuthFetch('/api/chat/masks/', {
        method: 'POST',
        body: {
            title: props.maskTitle[0],
            avatar: props.maskTitle[1],
            mask: JSON.stringify(props.fewShotMessages)
        }
    })
    submittingNewMask.value = false
    menu.value = false
}

const adjustTextAreaHeightWhenFocus = (event, idx) => {
    props.showButtonGroup[idx] = false
    const textarea = event.target
    textarea.rows = 5
}

const adjustTextAreaHeightWhenBlur = (event, idx) => {
    props.showButtonGroup[idx] = true
    const textarea = event.target
    textarea.rows = 1
}

const setAvatar = (emoji) => {
    emit('updateAvatar', emoji.i)
    showEmojiPicker.value = false
}
</script>

<template>
    <div>
        <v-menu v-model="menu" :close-on-content-click="false">
            <template v-slot:activator="{ props: menu }">
                <v-tooltip location="top" :text="maskTitle[0]">
                    <template v-slot:activator="{ props: tooltip }">
                        <v-btn
                            v-bind="mergeProps(menu, tooltip)"
                            icon
                            :title="$t('presetFewShotMask')"
                            class="toolbar-btn"
                        >
                            <v-icon
                                :icon="
                                    fewShotMessages.length === 0
                                        ? 'fa:fa-solid fa-masks-theater'
                                        : 'fa:fa-solid fa-mask'
                                "
                                style="padding-bottom: 2px"
                                fade
                            ></v-icon>
                        </v-btn>
                    </template>
                </v-tooltip>
            </template>

            <v-container class="card-size card-custom">
                <v-card>
                    <v-toolbar
                        density="compact"
                        class="d-flex justify-between-spacing"
                        :height="isMobile ? '56' : '64'"
                    >
                        <v-toolbar-title class="headline">{{
                            $t('presetFewShotMask')
                        }}</v-toolbar-title>
                    </v-toolbar>

                    <v-divider></v-divider>

                    <v-list class="list-max-height" :disabled="submittingNewMask">
                        <div
                            v-if="fewShotMessages.length > 0"
                            class="pt-3 ml-1 mask-title-custom pd-custom"
                        >
                            <h3 style="margin: 0 10px 20px 0" :class="pfs.n">
                                {{ $t('maskTitle') }}
                            </h3>
                            <v-btn
                                icon
                                variant="outlined"
                                @click="showEmojiPicker = !showEmojiPicker"
                                class="avatar-btn"
                            >
                                <v-icon style="margin-bottom: 5px" :class="pfs.l">{{
                                    maskTitle[1]
                                }}</v-icon>
                            </v-btn>
                            <v-text-field
                                v-model="maskTitle[0]"
                                density="compact"
                                variant="outlined"
                            ></v-text-field>
                            <EmojiPicker
                                v-if="showEmojiPicker"
                                class="emoji-picker-custom"
                                :native="true"
                                @select="setAvatar"
                            ></EmojiPicker>
                            <v-spacer></v-spacer>
                        </div>
                        <template
                            v-for="(fewShotMessage, idx) in fewShotMessages"
                            :key="fewShotMessage.id"
                        >
                            <div class="pt-3 list-item-custom pd-custom">
                                <v-btn
                                    icon
                                    title="system"
                                    class="square"
                                    elevation="0"
                                    v-if="showButtonGroup[idx]"
                                    @click="fewShotMessage.role = 'system'"
                                    :color="fewShotMessage.role == 'system' ? 'primary' : ''"
                                >
                                    <v-icon icon="settings" size="24"></v-icon>
                                </v-btn>
                                <v-btn
                                    icon
                                    title="user"
                                    class="square"
                                    elevation="0"
                                    v-if="showButtonGroup[idx]"
                                    @click="fewShotMessage.role = 'user'"
                                    :color="fewShotMessage.role === 'user' ? 'primary' : ''"
                                >
                                    <v-icon icon="person" size="24"></v-icon>
                                </v-btn>
                                <v-btn
                                    icon
                                    title="assistant"
                                    class="square"
                                    elevation="0"
                                    v-if="showButtonGroup[idx]"
                                    @click="fewShotMessage.role = 'assistant'"
                                    :color="fewShotMessage.role === 'assistant' ? 'primary' : ''"
                                >
                                    <v-icon icon="smart_toy" size="24"></v-icon>
                                </v-btn>

                                <v-textarea
                                    rows="1"
                                    v-model="fewShotMessage.content"
                                    variant="outlined"
                                    density="compact"
                                    hide-details
                                    class="textarea-custom"
                                    :label="$t(`${fewShotMessage.role}Preset`)"
                                    :tabindex="idx + 1"
                                    v-on:focus="adjustTextAreaHeightWhenFocus($event, idx)"
                                    v-on:blur="adjustTextAreaHeightWhenBlur($event, idx)"
                                >
                                </v-textarea>

                                <v-btn
                                    icon
                                    title="delete"
                                    v-if="showButtonGroup[idx]"
                                    @click="deleteMessage(idx)"
                                    class="square"
                                    color="transparent"
                                    elevation="0"
                                >
                                    <v-icon icon="highlight_remove" size="24"></v-icon>
                                </v-btn>
                            </div>
                        </template>
                        <div ref="grab" class="w-100" style="height: 5px"></div>
                    </v-list>
                    <v-divider v-show="props.fewShotMessages.length > 0"></v-divider>
                    <div class="action-custom">
                        <v-btn
                            variant="outlined"
                            class="action-btn-custom"
                            :loading="submittingNewMask"
                            @click="saveFewShotMask()"
                        >
                            <v-icon
                                icon="fa:fa-solid fa-store"
                                :style="`font-size: ${isMobile ? 0.7 : 1}rem;`"
                            ></v-icon>
                            <span style="padding-left: 2px">{{ $t('favorite') }}</span>
                        </v-btn>
                        <v-btn variant="outlined" class="action-btn-custom" @click="menu = false">
                            <v-icon icon="save"></v-icon>
                            <span style="padding-left: 2px">{{ $t('save') }}</span>
                        </v-btn>
                        <v-spacer></v-spacer>
                        <v-btn
                            :disabled="submittingNewMask"
                            variant="outlined"
                            @click="addMessage()"
                            class="action-btn-custom"
                        >
                            <v-icon icon="add_circle_outline"></v-icon>
                            <span style="padding-left: 2px">{{ $t('addPresetFewShotMask') }}</span>
                        </v-btn>
                        <v-btn
                            :disabled="submittingNewMask"
                            variant="outlined"
                            class="action-btn-custom"
                            @click="clearFewShotMask()"
                        >
                            <v-icon icon="refresh"></v-icon>
                            <span style="padding-left: 2px">{{ $t('clear') }}</span>
                        </v-btn>
                    </div>
                </v-card>
            </v-container>
        </v-menu>
    </div>
</template>

<style scoped>
@media screen and (min-width: 851px) {
    .card-size {
        width: 800px;
    }
    .pd-custom {
        padding: 0 24px 0 24px;
    }
}
@media screen and (max-width: 850px) {
    .card-size {
        width: 500px;
    }
    .pd-custom {
        padding: 0 16px 0 16px;
    }
}
@media screen and (min-width: 501px) {
    .square {
        height: 36px;
        width: 36px;
        margin: 0 2px;
        border-radius: 5px !important;
    }
    .avatar-btn {
        margin: 0 20px 20px 0;
        height: 40px;
        width: 40px;
    }
}
@media screen and (max-width: 500px) {
    .toolbar-btn {
        font-size: 0.9rem;
    }
    .card-size {
        width: auto;
        min-width: 335px;
    }
    .pd-custom {
        padding: 0 4px 0 12px;
    }
    .square {
        height: 32px;
        width: 32px;
        margin: 0 1px;
        border-radius: 5px !important;
    }
    .avatar-btn {
        margin: 0 20px 20px 0;
        height: 35px;
        width: 35px;
    }
    .action-btn-custom {
        margin: 0 0px !important;
        padding: 0 10px;
        /* font-size: 0.8rem; */
        border: none;
    }
}
.card-custom {
    padding: 0;
    display: flex;
    flex-direction: column;
}
.list-max-height {
    max-height: 400px;
    overflow: auto;
    padding: 0 0 5px 0;
}
.mask-title-custom {
    display: flex;
    align-items: center;
}
.action-custom {
    display: flex;
    flex-direction: row-reverse;
    margin: 10px 10px;
}
.action-btn-custom {
    margin: 0 10px;
}
.list-item-custom {
    display: flex;
    align-items: center;
    justify-content: space-between;
}
.textarea-custom {
    flex-grow: 1;
    margin: 0 5px 0 10px;
}
.emoji-picker-custom {
    position: fixed;
    z-index: 9999;
    right: 48%;
    bottom: 1%;
}
</style>

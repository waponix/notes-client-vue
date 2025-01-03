<script setup lang="ts">
import { nextTick, onMounted, ref, type Ref } from 'vue'

import BaseLayout from '@/components/layouts/BaseLayout.vue'
import Tabs from '@/components/Tabs.vue'
import Request from '@/components/api/Request.vue'
import Convo from '@/components/Convo.vue'

import InputIcon from '@/components/fields/InputIcon.vue'
import InputMultiAction from '@/components/fields/InputMultiAction.vue'

import SearchIcon from '@/components/icons/SearchIcon.vue'
import PlusIcon from '@/components/icons/PlusIcon.vue'
import PinIcon from '@/components/icons/PinIcon.vue'
import StarIcon from '@/components/icons/StarIcon.vue'
import SendIcon from '@/components/icons/SendIcon.vue'
import SweepIcon from '@/components/icons/SweepIcon.vue'

import WriteLoaderAnimatedIcon from '@/components/icons/WriteLoaderAnimatedIcon.vue'

import { useNoteStore } from '@/stores/notes'
import router from '@/router'

const noteStore = useNoteStore()

const searchError: Ref<null|string> = ref(null) 
const userInput: Ref<string> = ref('')

const parseResponse = (response: any) => {
    // @ts-ignore
    noteStore.generalConvo.push(...[{
        role: 'user',
        content: userInput.value,
    }, {
        role: 'assistant',
        content: response.data.answer,
    }])

    noteStore.generalToken = response.data.token
    
    showConvoLoader.value = false
    setTimeout(() => {
        userInput.value = ''
        clearQueryInput()
        scrollToBottom()
    }, 250)
}

const showConvoLoader: Ref<boolean> = ref(false)

const scrollToBottom = () => {
    let convo = document.querySelector('#app-ai-convo')
    //@ts-ignore
    convo.scrollTop = convo?.scrollHeight
}

const clearQueryInput = () => {
    let input = document.querySelector('textarea#question')
    //@ts-ignore
    input.value = ''
}

const sendQuery = (request: any) => {
    request.send()
    showConvoLoader.value = true
    setTimeout(() => {
        scrollToBottom()
    }, 250)
}

const openNewNoteTab = () => {
    noteStore.openTab({
        id: 'untitled',
        name: 'Untitled',
        url: '/note/untitled',
    })

    router.push('/note/untitled')
}

const clearConvo = () => {
    noteStore.clearGeneralConvo()
}

onMounted(async() => {
    await nextTick()
    scrollToBottom()
})

</script>

<template>
    <BaseLayout>
        <template v-slot:header>
            <Tabs/>
        </template>
        <template v-slot:left-content>
            <div class="flex flex-col w-[70vw] px-[25px] mt-[25px]">
                <!-- search field and action buttons -->
                <div class="app-action-box flex items-center flex-initial w-full">
                    <InputIcon
                    id="app-note-search"
                    :error="searchError"
                    class="max-w-[400px]"
                    >
                        <SearchIcon />
                    </InputIcon>
                    <div class="app-action-buttons flex flex-initial px-[25px]">
                        <a @click.prevent="openNewNoteTab" href="">
                            <PlusIcon />
                        </a>
                        <a @click.prevent href="">
                            <PinIcon />
                        </a>
                        <a @click.prevent href="">
                            <StarIcon />
                        </a>
                    </div>
                </div>
                 <!-- main content -->
                <div class="flex-1 grid gap-[25px] grid-cols-3 pt-[25px]">
                    
                </div>
            </div>
        </template>
        <template v-slot:right-content>
            <div class="flex flex-col w-[30vw] px-[25px] my-[25px] border-l-[1px] border-normal">
                <div class="flex-1"></div>
                <!-- AI chatbox -->
                <PerfectScrollbar 
                id="app-ai-convo" 
                class="flex-initial max-h-[76vh] flex flex-col text-[14px] pr-[10px] overflow-x-hidden">
                    <Convo :data="noteStore.generalConvo" />
                    <div v-if="userInput" class="mb-[20px]">
                        <span
                        style="white-space: pre-line" 
                        class="float-end py-[5px] px-[20px] rounded-[10px] text-hazy-light bg-darker max-w-[80%]">
                            {{ userInput }}
                        </span>
                    </div>
                    <div v-if="showConvoLoader" class="pb-[20px] flex justify-center items-end text-hazy-light">
                        <WriteLoaderAnimatedIcon class="mr-[5px]"/>
                        <span>just a moment...</span>
                    </div>
                </PerfectScrollbar>
                <Request
                @success="parseResponse"
                name="assistant/notes/query"
                :data="{
                    query: userInput,
                    token: noteStore.generalToken
                }"
                method="POST"
                v-slot="request">
                    <InputMultiAction
                    @value="v => {
                        userInput = v
                        scrollToBottom()
                    }"
                    id="question"
                    class="flex-initial"
                    placeholder="Need assistance? Ask away!">
                        <a
                        @click.prevent="clearConvo" 
                        href="">
                            <SweepIcon />
                        </a>
                        <a
                        @click.prevent="sendQuery(request)" 
                        href="">
                            <SendIcon />
                        </a>
                    </InputMultiAction>
                </Request>
            </div>
        </template>
    </BaseLayout>
</template>

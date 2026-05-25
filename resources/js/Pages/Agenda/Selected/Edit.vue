<script setup>
import Errors from '@/Components/Alert/Errors.vue';
import Success from '@/Components/Alert/Success.vue';
import LinksButton from '@/Components/Button/LinksButton.vue';
import InputLabel from '@/Components/Input/InputLabel.vue';
import TextInput from '@/Components/Input/TextInput.vue';
import AppLayout from '@/Layouts/AppLayout.vue';
import ContentLayout from '@/Layouts/ContentLayout.vue';
import FormLayout from '@/Layouts/FormLayout.vue';
import { Head, router, useForm } from '@inertiajs/vue3';
import FileInput from '@/Components/Input/FileInput.vue';
import Selection from '@/Components/Input/Selection.vue';
import { capitalize } from 'vue';
import TextAreaInput from '@/Components/Input/TextAreaInput.vue';
import { agendaStatus } from '@/utils/status';

const props = defineProps({
    agenda: Object,
    auths: Object
})

const attachment = props.agenda.attachment
    ? {
          url: props.agenda.attachment.url,
          name: props.agenda.attachment.name
      }
    : null

const transForm = useForm({
    title: props.agenda.title,
    date: props.agenda.date,
    status: props.agenda.status,
    notes: props.agenda.notes,
    file_path: null
})

const handleFileUpload = (event) => {
    transForm.file_path = event.target.files[0]
}

const updateForm = () => {
    transForm
        .transform((data) => ({
            ...data,
            _method: 'put',
        }))
        .post({
            preserveScroll: true,
            forceFormData: true,
        })
}
</script>

<template>
    <Head title="Edit Agenda"/>

    <AppLayout>
        <ContentLayout>
            <template #content-head-buttons>
                <LinksButton @click="router.get(route('agenda.view', agenda.agenda_id))" :class="['bg-red-600']">
                    <i class="fa-solid fa-arrow-left"></i><span class="ml-2">Back</span>
                </LinksButton>
            </template>
            <template #main-content>
                <div class="grid grid-cols-1 lg:grid-cols-3 gap-2 overflow-y-auto">
                    <div class="p-3 col-span-2">
                        <Success />
                        <Errors />

                        <form @submit.prevent="updateForm">
                            <FormLayout form-title="Edit Agenda">
                                <template #fields>
                                    <div v-if="props.auths.isUser" class="col-span-2 mb-4 p-3 bg-yellow-100 text-yellow-800 text-sm rounded-lg">
                                       <i class="fa-solid fa-triangle-exclamation"></i> You have view-only access to this agenda.
                                    </div>

                                    <div>
                                        <InputLabel label-for="Title"/>
                                        <TextInput
                                            name="title"
                                            v-model="transForm.title"
                                            :disabled="!props.auths.isAdmin"
                                            required
                                            :class="['disabled:cursor-not-allowed']"
                                        />
                                    </div>

                                    <div>
                                        <InputLabel label-for="Date" />
                                        <TextInput
                                            type="date"
                                            name="date"
                                            v-model="transForm.date"
                                            :disabled="!props.auths.isAdmin"
                                            required
                                            :class="['disabled:cursor-not-allowed']"
                                        />
                                    </div>

                                    <div>
                                        <InputLabel label-for="Status" />
                                        <Selection v-if="props.auths.isAdmin"
                                            :initial-val-label="capitalize(transForm.status)"
                                            :initial-value="transForm.status"
                                            v-model="transForm.status"
                                        >
                                            <option
                                                v-for="status in agendaStatus"
                                                :key="status.value"
                                                :value="status.value"
                                            >
                                                {{ status.label }}
                                            </option>
                                        </Selection>
                                        <TextInput v-else
                                            :value="capitalize(transForm.status)"
                                            readonly
                                        />
                                    </div>

                                    <div class="col-span-2">
                                        <InputLabel label-for="Notes" />
                                        <TextAreaInput
                                            name="notes"
                                            v-model="transForm.notes"
                                            :disabled="!props.auths.isAdmin"
                                            :class="['disabled:cursor-not-allowed']"
                                        />
                                    </div>

                                    <div>
                                        <InputLabel label-for="Replace File" />
                                        <FileInput
                                            @change="handleFileUpload"
                                            :disabled="!props.auths.isAdmin"
                                            :class="['disabled:cursor-not-allowed']"
                                        />
                                        <p v-if="attachment" class="text-sm text-gray-600 mt-2">
                                            Current file:
                                            <a :href="attachment.url" target="_blank" class="text-blue-600 hover:text-blue-800 underline">
                                                {{ attachment.name }}
                                            </a>
                                        </p>

                                        <p v-if="transForm.errors.file_path" class="text-red-500 text-sm mt-1">
                                            {{ transForm.errors.file_path }}
                                        </p>
                                    </div>
                                </template>
                                <template #buttons>

                                </template>
                            </FormLayout>
                        </form>
                    </div>
                </div>
            </template>
        </ContentLayout>
    </AppLayout>
</template>
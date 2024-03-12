<script setup>
import {ref, onBeforeMount } from 'vue';
import axios from 'axios';
import Card from "./components/Card.vue";

const segments = ref([]);
const totalClients = ref(0);
const isLoading = ref(true);
const errorMessage = ref('');

const baseUrl = import.meta.env.VITE_VUE_APP_BASE_URL;

onBeforeMount(() => {
    fetchSegments();
});

// Сохранение токена в localStorage
function saveTokenToLocalStorage(token) {
    localStorage.setItem('auth_token', token);
}

// Получение токена из localStorage
function getTokenFromLocalStorage() {
    return localStorage.getItem('auth_token');
}

// Получение токена по api
async function fetchToken() {
    try {
        const data = new FormData();
        data.append('username', import.meta.env.VITE_VUE_APP_USERNAME);
        data.append('password', import.meta.env.VITE_VUE_APP_PASSWORD);

        const response = await axios.post(`${baseUrl}/user/token`, data, {
            headers: {
                'Content-Type': 'multipart/form-data',
            },
        });

        const token = response.data['access_token'];
        saveTokenToLocalStorage(token);
        return token;
    } catch (error) {
        console.log("Failed to fetch token:", error);
    }
}

// Получение id отчета по api
async function fetchReportId(token) {
    try {
        const reportsResponse = await axios.get(`${baseUrl}/reports`, {
            headers: {Authorization: `Bearer ${token}`}
        });
        const reports = reportsResponse.data.grouped.General;
        const report = reports.find(report => report.name === "get_segment_rfm");
        return report.id;
    } catch (error) {
        console.error("Failed to fetch report id:", error);
    }
}

// Получение данных по сегментам
async function fetchSegments() {
    isLoading.value = true;
    try {
        let token = getTokenFromLocalStorage();
        if (!token) {
            token = await fetchToken();
        }

        const reportId = await fetchReportId(token);

        // Получение сегментов
        const segmentsResponse = await axios.post(`${baseUrl}/report/${reportId}/run`, {}, {
            headers: {
                accept: 'application/json',
                Authorization: `Bearer ${token}`,
                'Content-Type': 'application/json'
            }
        });

        const segmentsData = segmentsResponse.data.aggregations.segments.buckets;
        segments.value = segmentsData;
       // segments.value = [];

        // Подсчет общего количества клиентов по данным из всех сегментов
        totalClients.value = segmentsData.reduce((total, segment) => total + segment.doc_count, 0);
    } catch (error) {
        errorMessage.value = 'Ошибка: не удалось получить сегменты!';
        console.error("Failed to fetch segments:", error);
    } finally {
        isLoading.value = false;
    }
}
</script>

<template>
    <div class="container flex flex-wrap justify-center gap-6">
        <!-- Индикатор загрузки -->
        <div v-if="isLoading" class="text-lg font-bold">Загрузка...</div>
        <div v-if="errorMessage" class="text-red-500 font-bold">{{ errorMessage }}</div>

        <!-- Отображение сегментов -->
        <div v-if="!isLoading && segments.length"
        class="flex flex-wrap">
            <Card
                v-for="segment in segments"
                :key="segment.key"
                :data="segment"
                :total="totalClients"
                class="mr-4 mb-4"
            />
        </div>
        <div v-else class="text-lg font-bold">Сегменты отсутствуют</div>

    </div>
</template>


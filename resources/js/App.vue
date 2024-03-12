<script setup>
import {ref, onMounted} from 'vue';
import axios from 'axios';
import Card from "./components/Card.vue";

const segments = ref([]);
const totalClients = ref(0);

const baseUrl = import.meta.env.VITE_VUE_APP_BASE_URL;

onMounted(() => {
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
        console.error("Failed to fetch token:", error);
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

async function fetchSegments() {
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

        // Подсчет общего количества клиентов по данным из всех сегментов
        totalClients.value = segmentsData.reduce((total, segment) => total + segment.doc_count, 0);
    } catch (error) {
        console.error("Failed to fetch segments:", error);
    }
}
</script>

<template>
    <div class="container flex flex-wrap justify-center gap-6">
        <Card v-for="segment in segments"
              :key="segment.key"
              :data="segment"
              :total="totalClients"
        />
    </div>
</template>


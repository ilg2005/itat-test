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
async function fetchSegments() {
    try {
        const token = getTokenFromLocalStorage();
        if (!token) {
            // Получение токена авторизации
            const data = new FormData();
            data.append('username', import.meta.env.VITE_VUE_APP_USERNAME);
            data.append('password', import.meta.env.VITE_VUE_APP_PASSWORD);
            const authResponse = await axios.post(`${baseUrl}/user/token`, data, {
                headers: {
                    'Content-Type': 'multipart/form-data',
                },
            });
            const token = authResponse.data['access_token'];
            console.log(token);
            saveTokenToLocalStorage(token);
        }

        // Получение id отчета
        const reportsResponse = await axios.get(`${baseUrl}/reports`, {
            headers: {Authorization: `Bearer ${token}`}
        });
        const reports = reportsResponse.data.grouped.General;

        const report = reports.find(report => report.name === "get_segment_rfm");
        const reportId = report.id;

        console.log(reportId);

        // Получение сегментов
        const segmentsResponse = await axios.post(`${baseUrl}/report/${reportId}/run`, {}, {
            headers: {
                accept: 'application/json',
                Authorization: `Bearer ${token}`,
                'Content-Type': 'application/json'
            }
        });

        console.log(segmentsResponse.data);
        totalClients.value = segmentsResponse.data.total;
        console.log(totalClients.value);
        const segmentsData = segmentsResponse.data.aggregations.segments.buckets;
        console.log(segmentsData);
        segments.value = segmentsData;

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


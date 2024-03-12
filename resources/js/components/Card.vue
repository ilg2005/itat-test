<script setup>
import translations from "../../assets/translations.json";

const props = defineProps({
    data: Object,
    total: Number
});

const segment = {
    title: translations[props.data.key].translation,
    clientsPercent: Math.round((props.data.doc_count / props.total) * 100),
    description: translations[props.data.key].description,
    clientsNumber: props.data.doc_count,
    ordersTotal: Math.round(props.data.total.value),
    averageOrder: Math.round(props.data.total.value / props.data.orders.value),
    color: translations[props.data.key].color
}
</script>

<template>
    <div class="max-w-[350px] p-4 flex flex-col bg-white rounded-lg shadow-md">

        <div class="flex justify-between items-center mb-2 font-bold">
            <h2 class="text-lg">{{ segment.title }}</h2>
            <span :style="{ backgroundColor: segment.color }"
                  class="custom text-sm text-white">
                Всего: {{ segment.clientsPercent }}%
            </span>
        </div>

        <p class="text-gray-400 ">{{ segment.description }}</p>

        <div class="mt-4 space-y-3">
            <div class="custom inline-flex bg-gray-300">
                <div>Кол-во клиентов: <span class="font-bold">{{ segment.clientsNumber }}</span></div>
            </div>
            <div class="custom inline-flex bg-gray-300">
                <p>Сумма заказов: <span class="font-bold">{{ segment.ordersTotal }} ₽</span></p>
            </div>
            <div class="custom inline-flex bg-gray-300">
                <p>Средний чек: <span class="font-bold">{{ segment.averageOrder }} ₽</span></p>
            </div>
        </div>
    </div>
</template>

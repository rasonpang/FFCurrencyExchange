<template>
	<span class="currency-input">
		<Multiselect
			class="multi-select"
			v-model="value.currency"
			:options="options.map((i) => i.code)"
			:allow-empty="false"
			:custom-label="customLabel"
		>
			<template #singleLabel="props">
				{{ props.option }}
			</template>
		</Multiselect>
		<input
			class="currency-value"
			v-model="value.value"
			@focus="onFocus"
		/>
	</span>
</template>
<script setup>
import axios from "axios";
import { onMounted, ref, watch } from "vue";
import Multiselect from "vue-multiselect";

const props = defineProps(["value"]);
const emit = defineEmits(["onFocus"]);

const options = ref([]);

const customLabel = (value) => {
	const option = options.value.find((i) => i.code == value) ?? {
		name: "",
		code: "",
	};
	return `${option.name} (${option.code})`;
};

const getCurrencies = async () => {
	const result = await axios.get(
		"https://api.frankfurter.app/currencies"
	);

	if (result.status == 200) {
		options.value = Object.entries(result.data).reduce(
			(prev, [code, name]) => {
				prev.push({ code, name });
				return prev;
			},
			[]
		);
	}
};

const onFocus = () => {
	emit("onFocus");
};

onMounted(async () => {
	await getCurrencies();
});
</script>
<style scoped>
.option::before {
	background-image: url(https://upload.wikimedia.org/wikipedia/commons/thumb/7/7a/Flag_of_the_Republic_of_Abkhazia.svg/23px-Flag_of_the_Republic_of_Abkhazia.svg.png);
}
</style>

<template>
	<div>
		<span class="input-container">
			<span>
				<CurrencySelect
					:value="source"
					@onValueChange="onValueChange('source')"
				/>
			</span>

			<span>
				<img
					class="switch-button"
					src="./assets/switch.svg"
					alt="Switch"
					@click="switchCurrency"
				/>
			</span>

			<span>
				<CurrencySelect
					:value="target"
					@onValueChange="onValueChange('target')"
				/>
			</span>
		</span>
	</div>
</template>
<script setup>
import { ref, watch } from "vue";
import CurrencySelect from "./components/CurrencySelect.vue";
import axios from "axios";

const source = ref({
	currency: "USD",
	value: 0,
});

const target = ref({
	currency: "USD",
	value: 0,
});

const currencyRate = ref({});

let isWaiting = false;

const throttle = (func, wait = 250) => {
	return (...args) => {
		if (!isWaiting) {
			func.apply(this, args);
			isWaiting = true;
			setTimeout(() => {
				isWaiting = false;
			}, wait);
		}
	};
};

const clone = (value) => JSON.parse(JSON.stringify(value));

const switchCurrency = () => {
	const from = clone(source.value);
	const to = clone(target.value);

	source.value = to;
	target.value = from;

	onValueChange("source");
};

const onValueChange = (key) => {
	const c = throttle(() => {
		const ground = key == "source" ? source : target;
		const opposite = key == "source" ? target : source;

		if (ground.value.currency == opposite.value.currency) {
			opposite.value.value = ground.value.value;
		} else {
			const result =
				ground.value.value *
				currencyRate.value[opposite.value.currency];
			opposite.value.value = result;
		}
	});
	c();
};

const getCurrencyRate = async () => {
	const result = await axios.get(
		`https://api.frankfurter.app/latest?from=${source.value.currency}`
	);

	if (result.status == 200) {
		currencyRate.value = result.data.rates;
	}
};

watch(() => source.value.currency, getCurrencyRate);

// https://www.frankfurter.app/docs/#historical
</script>

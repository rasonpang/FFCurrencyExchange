<template>
	<div>
		<div class="input-container">
			<div>
				<CurrencySelect
					:value="source"
					@onFocus="setFocusField('source')"
				/>
			</div>

			<div style="text-align: center; margin: 1em 0">
				<img
					class="switch-button"
					src="./assets/switch.svg"
					alt="Switch"
					@click="switchCurrency"
				/>
			</div>

			<div>
				<CurrencySelect
					:value="target"
					@onFocus="setFocusField('target')"
				/>
			</div>

			<div style="text-align: center; margin-top: 1em">
				({{ source.currency }}) TO ({{
					target.currency
				}}) =
				{{ currencyRate[target.currency] ?? 1 }}
			</div>
		</div>
	</div>
</template>
<script setup>
import { onMounted, ref, watch } from "vue";
import CurrencySelect from "./components/CurrencySelect.vue";
import axios from "axios";
import { values } from "lodash";

const focusedInput = ref("source");
const source = ref({
	currency: "USD",
	value: 0,
});
const target = ref({
	currency: "USD",
	value: 0,
});

const currencyRate = ref({});

const clone = (value) => {
	const parsedValue = JSON.parse(JSON.stringify(value));
	parsedValue.value = Number(parsedValue.value);
	return parsedValue;
};

const switchCurrency = async () => {
	const from = clone(source.value);
	const to = clone(target.value);

	source.value = to;
	target.value = from;

	await refreshData();

	// Make value accurate
	source.value.value = to.value;
};
const refreshData = async () => {
	focusedInput.value = "source";
	await getCurrencyRate();

	onValueChange();
};

const setFocusField = (key) => {
	focusedInput.value = key;
};

const onValueChange = () => {
	const focusingField = focusedInput.value;
	// If currency is same
	if (source.value.currency == target.value.currency) {
		to.value.value = from.value.value;
	} else {
		if (focusingField == "source") {
			const result =
				Math.round(
					(source.value.value *
						currencyRate.value[
							target.value.currency
						] +
						Number.EPSILON) *
						100
				) / 100;
			target.value.value = result;
		} else if (focusingField == "target") {
			const result =
				Math.round(
					(target.value.value /
						currencyRate.value[
							target.value.currency
						] +
						Number.EPSILON) *
						100
				) / 100;

			source.value.value = result;
		}
	}
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

watch(() => target.value.value, onValueChange);
watch(() => source.value.value, onValueChange);

onMounted(async () => {
	const data = localStorage.getItem("inputs");

	if (data) {
		const parsedCookie = JSON.parse(data);

		const newSource = clone(parsedCookie.source);
		const newTarget = clone(parsedCookie.target);

		source.value = newSource;
		target.value = newTarget;

		await refreshData();
	}

	// Make sure to save before closing
	window.onunload = () => {
		localStorage.setItem(
			"inputs",
			JSON.stringify({
				source: {
					currency: source.value.currency,
					value: source.value.value,
				},
				target: {
					currency: target.value.currency,
					value: target.value.value,
				},
			})
		);
	};
});
</script>

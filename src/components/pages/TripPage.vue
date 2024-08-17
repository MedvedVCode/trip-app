<template>
	<section class="trip">
		<h1 class="title">Друганы тратят</h1>
		<section>
			<div class="add-user">
				<button
					class="round-btn"
					@click="openModal"
				>
					+
				</button>
				<span class="add-user__text">добавь другана</span>
			</div>
			<modal-user-add
				ref="modal"
				@newUser="setNewUser"
			/>
			<div class="user">
				<button
					v-for="user in trip"
					:key="user.id"
					@click="selectUser(user)"
					class="user__name"
				>
					{{ user.name }}
				</button>
			</div>
			<p
				v-if="currentBro"
				class="active-bro"
			>
				{{ currentBro.name }} банкует
			</p>
			<div
				class="spread-to-all"
				v-if="currentBro"
			>
				<p>Разделить сумму на всех</p>
				<input
					type="number"
					class="money__spent"
					v-model="currentBro.spentInTime"
				/>
				<button
					class="round-btn money__btn"
					@click="spreadCashToAll(payBro)"
				>
					&#10004;
				</button>
			</div>
			<ul class="money">
				<li
					class="money__users"
					v-for="users in payBro"
					:key="users.id"
				>
					<span class="money__name">{{ users.name }}</span>
					<input
						type="number"
						v-model="users.spentInTime"
						class="money__spent"
					/>
					<button
						class="round-btn money__btn"
						@click="setCash(users)"
					>
						&#10004;
					</button>
				</li>
			</ul>
			<ul class="result">
				<li
					v-for="result in resultBro"
					:key="result"
					class="result__item"
				>
					<span class="result__name">{{ result.firstBro }}</span>
					<span v-if="result.total < 0"> должен </span>
					<span v-else> получит от </span>
					<span class="result__name">{{ result.secondBro }}</span>
					<span class="result__cash">{{ Math.abs(result.total) }}</span>
				</li>
			</ul>
		</section>
		<button
			class="clear-btn"
			@click="clearAll"
			v-if="maxId > 0"
		>
			Очистить все
		</button>
	</section>
</template>

<script>
import ModalUserAdd from '@/components/details/ModalUserAdd.vue';
export default {
	name: 'TripPage',
	components: {
		ModalUserAdd,
	},
	data() {
		return {
			LOCAL_KEY: 'tripApp',
			trip: [],
			maxId: 0,
			selectedBro: {},
			spent: 0,
			totalResult: [],
		};
	},
	mounted() {
		this.trip = JSON.parse(localStorage.getItem(this.LOCAL_KEY)) || [];
		this.maxId = this.trip.reduce(
			(max, item) => (max < item.id ? item.id : max),
			0
		);
		this.selectedBro = this.trip[0];
		this.setTotalResult();
	},
	methods: {
		openModal() {
			this.$refs.modal.showModal();
		},
		setNewUser(data) {
			this.maxId++;
			const newName = {
				id: this.maxId,
				name: data,
				balance: [],
				spentInTime: 0,
			};
			this.trip.push(newName);
			this.selectedBro = this.trip[0];
			this.setTotalResult();
			localStorage.setItem(this.LOCAL_KEY, JSON.stringify(this.trip));
		},
		selectUser(user) {
			this.currentBro.spentInTime = 0;
			this.selectedBro = user;
			this.setTotalResult();
		},
		setCash(user) {
			this.selectedBro.balance.push({ id: user.id, cash: user.spentInTime });
			user.spentInTime = 0;
			const index = this.trip.indexOf((item) => item === this.selectedBro);
			this.trip[index] = this.selectedBro;
			this.setTotalResult();
			localStorage.setItem(this.LOCAL_KEY, JSON.stringify(this.trip));
		},
		spreadCashToAll(users) {
			const value = Math.floor(this.selectedBro.spentInTime / this.maxId);
			this.selectedBro.spentInTime = 0;
			users.forEach((user) => {
				user.spentInTime = value;
				this.setCash(user);
			});
		},
		setTotalResult() {
			const array = [];
			let resultItem = {};
			for (let i = 0; i < this.trip.length; i++) {
				for (let j = i + 1; j < this.trip.length; j++) {
					resultItem.total =
						this.getBalanceById(this.trip[i].balance, j + 1) -
						this.getBalanceById(this.trip[j].balance, i + 1);
					resultItem.firstBro = this.trip[i].name;
					resultItem.secondBro = this.trip[j].name;
					array.push(resultItem);
					resultItem = {};
				}
			}
			this.totalResult = array;
		},
		getBalanceById(bro, index) {
			let sum = 0;
			for (let balance of bro) {
				if (balance.id === index) {
					sum += balance.cash;
				}
			}
			return sum;
		},
		clearAll() {
			this.trip = [];
			localStorage.setItem(this.LOCAL_KEY, JSON.stringify(this.trip));
			this.maxId = 0;
			this.spent = 0;
			this.totalResult = [];
			this.selectedBro = undefined;
		},
	},
	computed: {
		payBro() {
			return this.trip.filter((item) => item !== this.selectedBro);
		},
		resultBro() {
			return this.totalResult;
		},
		currentBro() {
			return this.selectedBro;
		},
	},
};
</script>

<style lang="scss" scoped>
.trip {
	min-width: 150px;
	max-width: 600px;
	width: 100%;
	margin: 0 auto;
	border: 2px solid $accentColor;
	border-radius: 20px;
	padding: 30px;
}
.title {
	@include dmSerif(20px, 150%, 0.1px);
	color: $titleTxtColor;
	text-transform: uppercase;
}
.add-user {
	margin-top: 10px;
	display: flex;
	gap: 10px;
	@include jost(16px, 400, 29px, 0.16px);
	&__text {
		color: $textColor;
	}
}
.user {
	margin-top: 20px;
	display: flex;
	flex-wrap: wrap;
	gap: 10px;

	&__name {
		@include dmSerif(14px, 125%, 0.5px);
		color: $textColor;
		padding: 10px;
		background-color: $accentBGColor;
		border-radius: 10px;
		cursor: pointer;
		&:hover {
			background-color: $titleTxtColor;
			color: white;
		}
	}
}
.active-bro {
	margin-top: 20px;
	@include jost(16px, 400, 20px, 0.16px);
	color: $accentColor;
}
.spread-to-all {
	display: flex;
	gap: 20px;
	margin-top: 10px;
	align-items: center;
}
.money {
	margin-top: 10px;
	display: flex;
	gap: 10px;
	flex-direction: column;
	&__users {
		display: flex;
		gap: 20px;
	}
	&__name {
		@include jost(14px, 400, 20px, 0.16px);
		max-width: 100px;
		width: 100%;
		align-self: center;
		color: $textColor;
	}
	&__spent {
		width: 50px;
		padding-left: 5px;
		border-radius: 5px;
		border: 2px solid $accentColor;
		color: $textColor;
		align-self: center;
		@include jost(14px, 400, 20px, 0.16px);
	}
	&__btn {
		align-self: center;
	}
}

.result {
	margin-top: 20px;
	display: flex;
	flex-direction: column;
	gap: 5px;
	@include jost(14px, 400, 20px, 0.16px);
	color: $textColor;
	&__item {
		margin-top: 10px;
	}
	&__name {
		color: $accentColor;
	}
	&__cash {
		margin-left: 10px;
		@include dmSerif(14px, 100%, 0.5px);
		padding: 4px;
		background-color: $accentBGColor;
		border-radius: 5px;
	}
}

.clear-btn {
	margin-top: 20px;
	padding: 6px;
	border-radius: 10px;
	border: 2px solid $accentColor;
	height: 30px;
	background-color: #fff;
	color: $titleTxtColor;
	cursor: pointer;
	&:hover {
		color: $accentColor;
	}
}
</style>

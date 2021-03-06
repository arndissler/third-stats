<template>
	<div
		id='options'
		:class='{
			"dark": options.dark,
			"light": !options.dark,
			"text-normal": true,
			"background-normal": options.dark,
			"background-highlight-contrast": !options.dark,
		}'
	>
		<div class='container p-1'>
			<!-- section related to ThirdStats appearance and UI -->
			<h2>{{ $t('options.headings.appearance') }}</h2>
			<!-- option: dark -->
			<section class='entry'>
				<label for='dark'>
					{{ $t('options.darkMode.label') }}
					<span class="d-block text-gray text-small">{{ $t('options.darkMode.description') }}</span>
				</label>
				<div class='action'>
					<label class='switch'>
						<input type='checkbox' id='dark' v-model='options.dark' />
						<span class='switch-label' :data-on='$t("options.switch.on")' :data-off='$t("options.switch.off")'></span> 
						<span class='switch-handle'></span> 
					</label>
				</div>
			</section>
			<!-- section related to the stats page -->
			<h2 class='mt-3'>{{ $t('options.headings.stats') }}</h2>
			<!-- option: startOfWeek -->
			<section class='entry'>
				<label for='start'>
					{{ $t('options.startOfWeek.label') }}
					<span class='d-block text-gray text-small'>{{ $t('options.startOfWeek.description') }}</span>
				</label>
				<div class='action d-flex'>
					<select class='flex-grow' v-model='options.startOfWeek' id='start'>
						<option v-for='(name, pos) in weekdayNames' :key='pos' :value='pos'>{{ name }}</option>
					</select>
				</div>
			</section>
			<!-- option: addresses -->
			<section class='entry'>
				<label for='local'>
					{{ $t('options.localIdentities.label') }}
					<span class='d-block text-gray text-small'>{{ $t('options.localIdentities.description') }}</span>
				</label>
				<div class='action'>
					<div class="d-flex">
						<input  class='flex-grow' type='email' v-model='input.address' placeholder='hello@devmount.de' id='local' />
						<button @click='addAddress' class='button-inline'>
							<svg class="icon icon-small icon-bold d-block m-0-auto" viewBox="0 0 24 24">
								<path stroke="none" d="M0 0h24v24H0z" fill="none"/>
								<line x1="12" y1="5" x2="12" y2="19" />
								<line x1="5" y1="12" x2="19" y2="12" />
							</svg>
						</button>
					</div>
					<div>
						<span class='tag text-tiny' v-for='a in addressList'>
							{{ a }}
							<svg @click='removeAddress(a)' class="icon icon-bold icon-text cursor-pointer" viewBox="0 0 24 24">
								<path stroke="none" d="M0 0h24v24H0z" fill="none"/>
								<line x1="18" y1="6" x2="6" y2="18" />
								<line x1="6" y1="6" x2="18" y2="18" />
							</svg>
						</span>
					</div>
				</div>
			</section>
			<!-- option: account selection -->
			<section class='entry'>
				<label>
					{{ $t('options.activeAccounts.label') }}
					<span class='d-block text-gray text-small'>{{ $t('options.activeAccounts.description') }}</span>
				</label>
				<div class='action'>
					<div v-for='a in allAccounts' :key='a.id'>
						<label class="checkbox">
							<input type='checkbox' :value='a.id' v-model='options.accounts' />
							<i class="checkbox-icon"></i> {{ a.name }}
						</label>
					</div>
				</div>
			</section>
			<!-- section related to store processed data -->
			<h2 class='mt-3'>{{ $t('options.headings.storage') }}</h2>
			<!-- option: cache -->
			<section class='entry'>
				<label for='cache'>
					{{ $t('options.cache.label') }}
					<span class="d-block text-gray text-small">{{ $t('options.cache.description') }}</span>
				</label>
				<div class='action'>
					<label class='switch'>
						<input type='checkbox' id='cache' v-model='options.cache' />
						<span class='switch-label' :data-on='$t("options.switch.on")' :data-off='$t("options.switch.off")'></span> 
						<span class='switch-handle'></span> 
					</label>
				</div>
			</section>
			<!-- action: clear cache -->
			<section class='entry'>
				<label>
					{{ $t('options.clearCache.label') }}
					<span class="d-block text-gray text-small">{{ $t('options.clearCache.description') }}</span>
				</label>
				<div class='action'>
					<button @click='clearCache' class='mb-1'>{{ $t('options.button.clearCache') }}</button>
					<div class='text-gray text-small'>
						<span v-if='cacheSize > 0'>{{ $t('options.clearCache.size', [formattedCacheSize]) }}</span>
						<span v-else>{{ $t('options.clearCache.empty') }}</span>
					</div>
				</div>
			</section>
			<hr class='my-3' />
			<footer>
				<h3 class='text-thin mb-05'>{{ $t('options.note.title') }}</h3>
				<div class='text-gray text-small'>{{ $t('options.note.reloadStatsPage') }}</div>
			</footer>
		</div>
	</div>
</template>

<script>
import { formatBytes } from './utils'
export default {
	name: 'Options',
	data () {
		return {
			input: {
				address: '',
			},
			options: {
				dark: true,
				startOfWeek: 0,
				addresses: '',
				accounts: [],
				cache: true,
			},
			allAccounts: [],
			cacheSize: -1
		}
	},
	created: async function () {
		// initially load settings
		await this.getSettings()
		// initially load accounts
		this.getAccounts()
		// initially load cache size
		this.getCacheSize()
	},
	methods: {
		// create options object with given values or default values
		optionsObject (dark, startOfWeek, addresses, accounts, cache) {
			return {
				options: {
					dark: dark === null ? this.options.dark : dark,
					startOfWeek: startOfWeek === null ? this.options.startOfWeek : startOfWeek,
					addresses: addresses === null ? this.options.addresses : addresses,
					accounts: accounts === null ? this.options.accounts : accounts,
					cache: cache === null ? this.options.cache : cache,
				}
			}
		},
		// get all saved add-on settings
		getSettings: async function () {
			let result = await messenger.storage.local.get('options')
			// only load options if they have been set, otherwise default settings will be kept
			if (result && result.options) {
				this.options = result.options
			}
		},
		// get all existing accounts
		getAccounts: async function () {
			let accounts = await (await messenger.runtime.getBackgroundPage()).messenger.accounts.list()
			this.allAccounts = accounts
			// default accounts activated are all non local accounts ...
			if (!this.options.accounts.length) {
				accounts.map(a => {
					if (a.type != 'none') this.options.accounts.push(a.id)
				})
			}
			// unless there is only one local account
			if (this.options.accounts.length == 1 && this.options.accounts[0].type == 'none') {
				this.options.accounts.push(this.options.accounts[0].id)
			}
		},
		// get size of all cached account data
		getCacheSize: async function () {
			let allEntriesSize = new TextEncoder().encode(
				Object.entries(await messenger.storage.local.get())
					.map(([key, value]) => key + JSON.stringify(value))
					.join('')
			).length
			let optionsSize = new TextEncoder().encode(
				Object.entries(await messenger.storage.local.get('options'))
					.map(([key, value]) => key + JSON.stringify(value))
					.join('')
			).length
			this.cacheSize = allEntriesSize - optionsSize
		},
		// add configured email address to list of addresses and save
		addAddress: async function () {
			if (this.input.address) {
				let addresses = this.options.addresses ? this.options.addresses + ',' : ''
				addresses += this.input.address
				await messenger.storage.local.set(this.optionsObject(null, null, addresses, null, null))
				this.options.addresses = addresses
				this.input.address = ''
			}
		},
		// remove given email address from list of addresses and save
		removeAddress: async function (address) {
			let addresses = this.options.addresses.replace(address, '')
			addresses = addresses.replace(/,,/g, ',')
			addresses = addresses.replace(/^,+|,+$/g, '');
			await messenger.storage.local.set(this.optionsObject(null, null, addresses, null, null))
			this.options.addresses = addresses
		},
		// clear all cached stats entries
		clearCache: async function () {
			// clear whole local storage
			await messenger.storage.local.clear()
			// restore options
			await messenger.storage.local.set(this.optionsObject(null, null, null, null, null))
			// recalculate cache size
			this.getCacheSize()
		}
	},
	computed: {
		// array of localized, short day of week names
		weekdayNames () {
			let names = []
			for (let wd = 1; wd <= 7; wd++) {
				let d = new Date(1970, 1, wd) // choose a date to retrieve weekdays from, starting on a Sunday
				names.push(d.toLocaleDateString(this.$i18n.locale, { weekday: 'long' }))
			}
			return names
		},
		// array of email addresses configured for local account identities
		addressList () {
			return this.options.addresses ? this.options.addresses.split(',') : []
		},
		formattedCacheSize () {
			return formatBytes(this.cacheSize)
		}
	},
	watch: {
		options: {
			handler: function () {
				messenger.storage.local.set(this.optionsObject(null, null, null, null, null))
			},
			deep: true,
			immediate: false
		}
	}
}
</script>

<style lang='stylus'>
@require "assets/global"

// general
html, body
	min-height: 300px

// layout
#options
	width 100%
	height 100%

	.container > h2
		font-weight: 300

	.entry
		display: grid
		grid-template-columns: 1fr 1fr
		column-gap: 2rem
		margin-bottom: 1rem

		.action
			align-self: center

</style>

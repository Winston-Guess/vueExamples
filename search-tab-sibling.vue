<template>
	<div class="wmm-search-tab-pane"
	     style="max-width: 1000px;">
		<!-- Search inputs -->
		<el-form class="search-inputs-container"
		         label-position="left">
			<el-form-item v-for="(searchFieldKey) in searchInputs"
			              :key="searchFieldKey.label"
			              :label="tt(searchFieldKey.label)">
				<el-input v-model="searchFieldKey.searchString"
				          size="mini"
				          placeholder="Type to search"
				          v-on:input="searchOnTimeout"></el-input>
			</el-form-item>
		</el-form>
		<div class="fill-remaining-height">
			<div class="results-table-container">
				<!-- Search results table slot -->
				<slot></slot>
			</div>
		</div>
	</div>
</template>

<script lang='ts'>

	import { Form } from 'element-ui';
	import { SearchResultObject } from 'search/search';
	import { tt } from 'jsbase';
	import * as util from 'wmm/search/search-util';
	import "./search-tabs.less";

	export default {
		name: 'SearchInputs',

		components: {
			'el-form': Form
		},

		computed: {
			searchResultSync: {
				get() {
					return this.searchResultData;
				},
				set(val) {
					this.$emit('update:searchResultData', val);
				}
			},
			busySearchingSync: {
				get() {
					return this.busySearching;
				},
				set(val) {
					this.$emit('update:busySearching', val);
				}
			}
		},

		props: {
			searchInputs: {
				type: Object as () => util.ISearchInputFields
			},
			searchResultData: {
				type: Array as () => Object[] // can we use searchResultObj instead??
			},
			addRelatedRecords: {
				type: Boolean,
				default: false
			},
			busySearching: {
				type: Boolean,
				default: false
			}
		},

		data(): {
			timeout: Number;       // Multiple keypress timeout tracker
			tt: Function;
		} {
			return {
				timeout: null,
				tt: tt
			};
		},

		mounted() { },

		methods: {
			searchOnTimeout() {
				window.clearTimeout(this.timeout);
				this.timeout = window.setTimeout(() => {
					this.search();
				}, 400);
			},
			search() {
				let searchQueries = util.buildSearchQueries(this.searchInputs);
				if (searchQueries.length === 0) {
					this.searchResultSync = [];
					return;
				}

				const onSuccess = (searchResult: SearchResultObject) => {
					if (util.isRequestInFlight()) {
						return;
					} else if (!searchResult) {
						this.searchResultSync = [];
						this.busySearchingSync = false;
						return;
					}
					this.searchResultSync = util.extractData(searchResult);
					this.busySearchingSync = false;
				};

				this.busySearchingSync = true;
				util.search(searchQueries, onSuccess, this.addRelatedRecords);
			}
		}
	};

</script>

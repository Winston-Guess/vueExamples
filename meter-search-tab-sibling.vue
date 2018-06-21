<template>
	<search-inputs :searchInputs="searchInputs"
	               :searchResultData.sync="searchResultData"
	               :busySearching.sync="busySearching">
		<div class="results-table-container">
			<el-table v-loading="busySearching"
			          ref="resultTable"
			          :highlight-current-row="false"
			          :stripe="false"
			          :border="true"
			          :data="searchResultData"
			          max-height="auto"
			          @row-click="openMeterDetailsTab">
				<el-table-column prop="serial_no"
				                 :label="tt('Serial No')"
				                 width="160"></el-table-column>
				<el-table-column :label="tt('Address')">
					<el-table-column align="right"
					                 prop="street_number"
					                 width="70"></el-table-column>
					<el-table-column prop="street_name"
					                 width="200"></el-table-column>
					<el-table-column prop="suburb"></el-table-column>
				</el-table-column>
				<el-table-column prop="description"
				                 :label="tt('Status')"
				                 header-align="center"
				                 width="140">
					<template slot-scope="scope">
						<div class="tag-horizontal-flex">
							<el-tag :class="meterStatus[scope.row.description[0]]"
							        disable-transitions>{{tt(scope.row.description)}}</el-tag>
						</div>
					</template>
				</el-table-column>
			</el-table>
		</div>
	</search-inputs>
</template>

<script lang='ts'>

	import { Table, TableColumn, Tag } from 'element-ui';
	import { wmmEventBus, tabTypes, tabTypeMap } from 'wmm/wmm-util';
	import { ISearchInputFields, searchTypes } from 'wmm/search/search-util';
	import { tt } from 'jsbase';
	import SearchInputs from "wmm/search/search-tab.vue";
	import "./search-tabs.less";

	// Maps the meter status field values to its description and class color (see wmm-search-tabs.less)
	const meterStatus = {
		"N": "blue",
		"O": "gray",
		"I": "green",
		"R": "red",
		"S": "orange"
	};

	interface IStandData {
		searchInputs: ISearchInputFields;   // Data object that holds the search field strings
		searchResultData: Object[];              // Search result data object formatted to appear in element-ui table
		busySearching: boolean;                    // Tracks whether a search is in progress
		meterStatus: Object;                    // enum for meter status color class conversion
		tt: Function;
	}

	export default {
		name: 'MeterSearch',

		components: {
			'el-table': Table,
			'el-table-column': TableColumn,
			'el-tag': Tag,
			'search-inputs': SearchInputs
		},

		data(): IStandData {
			return {
				searchInputs: searchTypes.meter,
				searchResultData: [],
				busySearching: false,
				meterStatus: meterStatus,
				tt: tt
			};
		},

		mounted() { },

		methods: {
			openMeterDetailsTab(rowObject: Object) {
				wmmEventBus.createDetailsTab(tabTypes.meter, rowObject[tabTypeMap.meter.rangeField]);
			}
		}
	};

</script>

<template xmlns:v-bind="http://www.w3.org/1999/xhtml" xmlns:v-on="http://www.w3.org/1999/xhtml">
	<input
			id="{{uuid + 'input'}}"
			type="text"
			class="editable-mode form-control"
			:class="validationCssClass"
			v-if="editableMode" placeholder="{{ column }}"
			v-model="value"	
			v-on:keyup="autocomplete(column, uuid)"
	/>
	<template v-else>
		<a v-show="isUrl(value)" href="{{ value }}">{{ value }}</a>
		<span v-show="!isUrl(value)">{{ value }}</span>
	</template>
</template>

<script>
import Utils from './../../utils.js'

export default {
	props: {
		value: String,
		originalValue: String,		
		column: String,
		datatype: String,
		uuid: String,
		editableMode: {
			type: Boolean,
			default: true,
			required: false
		}
	},
	methods: {
		isUrl (data) {
			return Utils.isUrl(data)
		},
		parseLiteral (value) {
			let literalRegex = new RegExp(["^(\".*\")(\\^\\^xsd:(?:anyURI|boolean|date|dateTime|double|float|gDay|gMonth|",
					"gMonthDay|gYear|gYearMonth|integer|negativeInteger|nonNegativeInteger|nonPositiveInteger|positiveInteger|string|time))?"].join(''))
			let match = value.match(literalRegex)

			return match
		},
		autocomplete (column, uuid) {
			return Utils.autocomplete(column, uuid)
		}
	},
	computed: {
		isValueUpdated: function() {
			return this.value != this.originalValue
		},
		valueIsUrl: function() {
			return Utils.isUrl(this.value)
		},
		valueIsLiteral: function() {
			let match = this.parseLiteral(this.value)
			if (match) {
				return true
			}
			return false
		},
		valueIsLiteralWithDatatype: function() {
			let match = this.parseLiteral(this.value)
			if (match && match[2]) {
				return true
			}
			return false
		},
		validationCssClass: function() {
			if (!this.isValueUpdated) {
				return ''
			}

			if (this.datatype === 'uri') {
				if (this.valueIsUrl) {
					return 'valid'
				}
			} else if (this.datatype === 'literal') {
				if (this.valueIsLiteralWithDatatype) {
					return 'valid'
				} else if (this.valueIsLiteral) {
					return 'incomplete'
				}
			}
			return 'invalid'
		},
	},
	watch: {
		value (updatedValue) {
			this.$dispatch('updatedValue', this.column, updatedValue)
		}
	}
}
</script>
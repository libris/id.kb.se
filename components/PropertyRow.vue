<template>
  <div class="PropertyRow d-md-flex" :data-property="property">
    <span class="PropertyRow-bodyKey d-block d-md-inline" :title="translateKey(property)">{{ translateKey(property) }}</span>
    <span class="PropertyRow-bodyValue single" v-if="!Array.isArray(value)">
      <span class="" v-if="typeof value == 'boolean'">{{ translateUi(value == true ? 'Yes' : 'No') }}</span>
      <span v-else-if="typeof value !== 'object'">{{ value }}</span>
      <EntityNode :parent-key="property" :entity="value" v-else-if="!isByLangProperty" />
      <div class="PropertyRow-grid" v-else>
        <template v-for="(v, lang) in value">
          <div class="PropertyRow-gridKey" :key="`${property}-${lang}-key`">
            {{ lang }}
          </div>
          <div class="PropertyRow-gridValue"  :key="`${property}-${lang}-value`">
            {{ v }}
          </div>
        </template>
      </div>
    </span>
    <span class="PropertyRow-bodyValue multiple" v-if="Array.isArray(value)">
      <EntityNode :parent-key="property" :entity="node" v-for="(node, index) in finalizedValue" :key="index" />
    </span>
  </div>
</template>

<script>
import { mapGetters } from 'vuex';
import LensMixin from '@/mixins/lens';
import EntityNode from '@/components/EntityNode';
import * as DisplayUtil from 'lxltools/display';

export default {
  mixins: [LensMixin],
  data() {
    return {
      show: false,
      filteredTypes: [
        'Restriction',
      ],
    }
  },
  props: {
    property: {
      type: String,
      default: '',
    },
    value: {
      type: [Object, String, Number, Array, Boolean],
    },
  },
  methods: {
  },
  computed: {
    ...mapGetters(['entityReferences', 'settings', 'resources', 'vocabContext', 'display', 'vocab']),
    isByLangProperty() {
      return this.property.includes('ByLang');
    },
    containerType() {
      if (this.vocabContext[1].hasOwnProperty(this.property) && this.vocabContext[1][this.property].hasOwnProperty('@container')) {
        return this.vocabContext[1][this.property]['@container'];
      }
      return null;
    },
    finalizedValue() {
      return this.withoutFilteredTypes;
    },
    withoutFilteredTypes() {
      const original = this.valueSorted;
      return original.filter((item) => {
        if (item.hasOwnProperty('@type') == false) {
          return true;
        } else {
          return this.filteredTypes.includes(item['@type']) == false;
        }
      });
    },
    objectLabelReference() {
      // Returns a map with objects as keys and labels as values
      const refMap = new Map();
      this.value.forEach((item) => {
        const sortValue = typeof item === 'object' ? DisplayUtil.getItemLabel(
            item,
            this.resources,
            this.entityReferences,
            this.settings,
          ) : item;
        refMap.set(item, sortValue);
      });
      return refMap;
    },
    valueSorted() {
      if (this.containerType == '@set') {
        const ref = this.objectLabelReference;
        const value = this.value.slice(0);
        if (this.containerType == '@set') {
          return value.sort((a, b) => {
              if(ref.get(a) < ref.get(b)) { return -1; }
              if(ref.get(a) > ref.get(b)) { return 1; }
              return 0;
          });
        }
        return value;
      } else {
        return this.value;
      }
    },
  },
  components: {
    EntityNode,
  },
}
</script>

<style lang="scss">
.PropertyRow {
  border: solid $gray-200;
  border-width: 0px 0px 1px 0px;
  padding: 0.5rem 0;
  &:last-child {
    border-width: 0px;
  }

  &-grid {
    display: grid;
    grid-template-columns: 2em 1fr;
  }
  &-gridKey {
    font-family: monospace;
  }
  &-gridValue {

  }
  &-bodyKey {
    color: $gray-700;
    flex-basis: 15em;
    text-overflow: ellipsis;
    white-space: nowrap;
    overflow: hidden;
    font-size: 0.85rem;
    font-weight: 500;
    @media (min-width: 768px) {
      font-size: 1rem;
      font-weight: 400;
    }
    &:first-letter {
      text-transform: uppercase;
    }
  }
  &-bodyValue {
    flex-grow: 0;
    flex-basis: 100%;
    min-width: 0;
    @media (min-width: 768px) {
      padding-left: 1rem;
    }
    a {
      color: $kb-secondary-turquoise;
      text-decoration: none;
      word-break: break-all;
      &:hover {
        text-decoration: underline;
      }
    }
  }
}
</style>

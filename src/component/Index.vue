<template>
  <q-input :error="has_error" :model-value="number.valor" class="vue3-q-tel-input no-inherit-feedback" @update:model-value="phoneChanged" :maxlength="prev_value.length" :v-bind="$props">
    <template #prepend>
      <CountrySelection :use-icon="useIcon" :search-text="searchText" v-model:country="country" @countryChanged="countryChanged()" v-bind="dropdownOptions" class="no-border-field-before no-padding-field font-reduced-input-adon" />
    </template>
  </q-input>
</template>

<script lang="ts">
import CountrySelection from './CountrySelection.vue';
import { Country } from './types';
import { PhoneNumberUtil, PhoneNumber, PhoneNumberFormat } from 'google-libphonenumber';
import { getCountryByDialCode, getDefault, getCountryCodeFromPhoneNumber } from './countries';
import { QInput } from 'quasar';
import { defineComponent, ref, Ref } from 'vue';

const phoneNumberUtil: PhoneNumberUtil = new PhoneNumberUtil();

export default defineComponent({
  name: 'vue3-q-tel-input',
  components: {
    CountrySelection,
    QInput,
  },
  props: {
    tel: { type: Object, default: () => {}},
    required: { type: Boolean, default: () => false },
    searchText: { type: String, default: () => 'Search' },
    dropdownOptions: { type: Object, default: () => ({}) },
    defaultCountry: { type: String, default: () => 'us' },
    eagerValidate: { type: Boolean, default: () => true },
    useIcon: { type: Boolean, default: () => false },
  },
  emits: ['update:tel', 'input', 'error'],
  setup() {
    const country: Ref<Country> = ref(getDefault() as Country);
    const old_country: Ref<Country | undefined> = ref(undefined);
    const number: Ref<Object | any> = ref({iso:"", valor:""});
    const has_error: Ref<boolean> = ref(false);
    const prev_value: Ref<string> = ref('01234567890123456789');
    const phone_number: Ref<PhoneNumber | undefined> = ref(undefined);
    return {
      country,
      old_country,
      number,
      has_error,
      prev_value,
      phone_number,
    };
  },
  mounted() {
    this.country = getDefault(this.defaultCountry) as Country;
  },
  watch: {
    tel: {
      immediate: true,
      handler() {
        this.setPhone();
      },
    },
    defaultCountry: {
      immediate: true,
      handler() {
        if (this.defaultCountry) {
          this.country = getDefault(this.defaultCountry) as Country;
        }
      },
    },
    country: {
      immediate: true,
      handler() {
        this?.$nextTick(() => {
          this.old_country = this.country;
        });
      },
    },
  },
  methods: {
    getNumber(instance: PhoneNumber): string {
      if (!this.phone_number) return '';
      let phone: string = phoneNumberUtil.format(instance, PhoneNumberFormat.NATIONAL);
      if (phone.indexOf('0') === 0) phone = phone.replace(/^0/, '');
      return phone;
    },
    setPhone() {
      let country = this.country;
      if(this.tel.valor){
        if (this.tel.valor.toString() !== '') {
          const inCountry = getCountryCodeFromPhoneNumber(this.tel.valor.toString());
          if (inCountry && this.country.iso2 !== inCountry.iso2) {
            country = inCountry;
            this.$nextTick(() => {
              this.country = country;
            });
          }
        }
        try {
          this.phone_number = phoneNumberUtil.parse(this.tel.valor.toString().trim(), country.iso2);
          this.number.valor = this.getNumber(this.phone_number);
          this.has_error = !phoneNumberUtil.isValidNumberForRegion(this.phone_number, country.iso2);
        } catch (e) {
          this.phone_number = undefined;
          this.has_error = this.eagerValidate ? (this.tel.valor.toString().trim() === '' ? this.required : true) : false;
          this.number.valor = this.tel.valor?.toString().trim();
        }
      }
      this.$emit('error', this.has_error);
    },
    phoneChanged(val: string | number | null) {
      val = val === null ? '' : val.toString();
      let phone: PhoneNumber | undefined;
      try {
        phone = phoneNumberUtil.parse(val.trim(), this.country.iso2);
      } catch {
        phone = undefined;
      }
      const filtered_val = val.replace(/ /g, '');
      if (filtered_val.length > 2 && filtered_val.indexOf('+') === 0) {
        // some country code is in action
        const country = getCountryByDialCode(filtered_val);
        if (country) {
          this.country = country;
        }
      }
      const num = phone ? this.getNumber(phone) : val;
      this.prev_value = phone && phoneNumberUtil.isValidNumberForRegion(phone, this.country.iso2) ? this.getNumber(phone) : this.prev_value;//nos chupa un huevo

      if (num.replace(/ /g, '').length > this.prev_value.replace(/ /g, '').length) return this.setPhone(); // no need to update as its not valid
      this.$emit('update:tel', phone ? {iso: this.country.iso2 , valor: phoneNumberUtil.format(phone, PhoneNumberFormat.INTERNATIONAL) } : {
        iso:"",
        valor: ""
      });
      this.$emit('input', phone ? { iso: this.country.iso2 , valor: phoneNumberUtil.format(phone, PhoneNumberFormat.INTERNATIONAL) } : {
        iso:"",
        valor: ""
      });
    },
    countryChanged() {
      this.prev_value = '01234567890123456789';
      const value = {
        iso: this.country.iso2,
        valor: "",
      }
      this.number = value;
      this.phoneChanged(this.old_country ? value.valor.replace(`+${this.old_country.dialCode}`, `+${this.country.dialCode}`) : value.valor);
      // this.setPhone();
    },
  },
});
</script>

<style lang="scss">
@import '../style';
</style>

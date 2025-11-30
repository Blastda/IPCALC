
<script setup lang="ts">
import { ref, computed } from 'vue'
import { ButtonGroup, IpInput, MaskSelect, ResultCard } from 'packetcalculator'

const options = [
  "255.255.255.255", "255.255.255.254", "255.255.255.252", "255.255.255.248", "255.255.255.240", "255.255.255.224", "255.255.255.192", "255.255.255.128",
  "255.255.255.0", "255.255.254.0", "255.255.252.0", "255.255.248.0", "255.255.240.0", "255.255.224.0", "255.255.192.0", "255.255.128.0",
  "255.255.0.0", "255.254.0.0", "255.252.0.0", "255.248.0.0", "255.240.0.0", "255.224.0.0", "255.192.0.0", "255.128.0.0",
  "255.0.0.0", "254.0.0.0", "252.0.0.0", "248.0.0.0", "240.0.0.0", "224.0.0.0", "192.0.0.0", "128.0.0.0", "0.0.0.0"
]
const ip = ref('')
const mask = ref('255.255.255.0')
const isShowResult = ref(false)

const isIpValid = computed(() => {
  if (!ip.value) return false;
  if (!/^(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})$/.test(ip.value)) return false;
  const octets = ip.value.split('.').map(Number);
  return octets.every(octet => octet >= 0 && octet <= 255);
})

function getNetworkAddress(ip: string, mask: string): string {
  const ipOctets = ip.split('.').map(Number);
  const maskOctets = mask.split('.').map(Number);
  if (ipOctets.length !== 4 || maskOctets.length !== 4) return '';
  const result = [];
  for (let i = 0; i < 4; i++) {
    result.push((ipOctets[i] ?? 0) & (maskOctets[i] ?? 0));
  }
  return `${result[0]}.${result[1]}.${result[2]}.${result[3]}`;
}

function getAddressesCount(mask: string): number {
  let binaryMask = '';
  for (const octet of mask.split('.')) {
    binaryMask += Number(octet).toString(2).padStart(8, '0');
  }
  const ones = binaryMask.replace(/0/g, '').length;
  const zeros = 32 - ones;

  if (zeros >= 32) return 1;
  if (zeros === 31) return 2;
 

  return Math.pow(2, zeros) - 2;
}

function showResult() {
  if (isIpValid.value) {
    isShowResult.value = true;
  }
}

function resetForm() {
  ip.value = '';
  mask.value = '255.255.255.0';
  isShowResult.value = false;
}
</script>

<template>
  <div class="container">
    <h1>Калькулятор IP-подсетей</h1>
    <form class="form" @submit.prevent="showResult">
      <IpInput
        v-model="ip"
        :is-invalid="!!ip && !isIpValid"
        label="Введите IP-адрес"
        placeholder="Например: 192.168.1.100"
      />

      <MaskSelect
        v-model="mask"
        :options="options"
        label="Выберите маску подсети"
      />

      <ButtonGroup
        :is-disabled="!isIpValid"
        @submit="showResult"
        @reset="resetForm"
      />
    </form>

    <ResultCard
      :show="isShowResult && isIpValid"
      :ip-address="ip"
      :subnet-mask="mask"
      :network-address="getNetworkAddress(ip, mask)"
      :available-addresses="getAddressesCount(mask)"
    />
  </div>
</template>

<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #f5f7fa 0%, #e4edf5 100%);
  min-height: 100vh;
  padding: 20px;
}

.container {
  max-width: 600px;
  margin: 0 auto;
  background: white;
  border-radius: 16px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  padding: 30px;
}

h1 {
  text-align: center;
  color: #2c3e50;
  margin-bottom: 25px;
  font-size: 2rem;
}

.form {
  display: flex;
  flex-direction: column;
  gap: 20px;
  margin-bottom: 30px;
}

@media (max-width: 480px) {
  .container {
    padding: 20px;
  }

  h1 {
    font-size: 1.5rem;
  }
}
</style>

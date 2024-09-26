<template>
    <div>

      <p class="error">{{ error }}</p>

      <p class="decode-result">
        Last result: <b>{{ result }}</b>
      </p>

      <div>
        <qrcode-stream
            style="width: 100%; max-width: 600px; margin: 0 auto;"
          :constraints="selectedConstraints"
          :track="trackFunctionSelected.value"
          @error="onError"
          @detect="onDetect"
          @camera-on="onCameraReady"
        />
      </div>
    </div>
  </template>

  <script setup lang="ts">
  import { ref, computed } from 'vue'
  import { QrcodeStream } from 'vue-qrcode-reader'

  /*** detection handling ***/

  const result = ref('')

  function onDetect(detectedCodes) {
    console.log(detectedCodes)
    result.value = JSON.stringify(detectedCodes.map((code) => code.rawValue))
  }

  /*** select camera ***/

  const selectedConstraints = ref({ facingMode: 'environment' })
  const defaultConstraintOptions = [
    { label: 'rear camera', constraints: { facingMode: 'environment' } },
    { label: 'front camera', constraints: { facingMode: 'user' } }
  ]
  const constraintOptions = ref(defaultConstraintOptions)

  async function onCameraReady() {

    constraintOptions.value = [
      ...defaultConstraintOptions,
    ]

    error.value = ''
  }

  /*** track functons ***/

  function paintOutline(detectedCodes, ctx) {
    for (const detectedCode of detectedCodes) {
      const [firstPoint, ...otherPoints] = detectedCode.cornerPoints

      ctx.strokeStyle = 'red'

      ctx.beginPath()
      ctx.moveTo(firstPoint.x, firstPoint.y)
      for (const { x, y } of otherPoints) {
        ctx.lineTo(x, y)
      }
      ctx.lineTo(firstPoint.x, firstPoint.y)
      ctx.closePath()
      ctx.stroke()
    }
  }
  function paintBoundingBox(detectedCodes, ctx) {
    for (const detectedCode of detectedCodes) {
      const {
        boundingBox: { x, y, width, height }
      } = detectedCode

      ctx.lineWidth = 2
      ctx.strokeStyle = '#007bff'
      ctx.strokeRect(x, y, width, height)
    }
  }
  function paintCenterText(detectedCodes, ctx) {
    for (const detectedCode of detectedCodes) {
      const { boundingBox, rawValue } = detectedCode

      const centerX = boundingBox.x + boundingBox.width / 2
      const centerY = boundingBox.y + boundingBox.height / 2

      const fontSize = Math.max(12, (50 * boundingBox.width) / ctx.canvas.width)

      ctx.font = `bold ${fontSize}px sans-serif`
      ctx.textAlign = 'center'

      ctx.lineWidth = 3
      ctx.strokeStyle = '#35495e'
      ctx.strokeText(detectedCode.rawValue, centerX, centerY)

      ctx.fillStyle = '#5cb984'
      ctx.fillText(rawValue, centerX, centerY)
    }
  }
  const trackFunctionOptions = [
    { text: 'nothing (default)', value: undefined },
    { text: 'outline', value: paintOutline },
    { text: 'centered text', value: paintCenterText },
    { text: 'bounding box', value: paintBoundingBox }
  ]
  const trackFunctionSelected = ref(trackFunctionOptions[1])

  /*** error handling ***/

  const error = ref('')

  function onError(err) {
    error.value = `[${err.name}]: `

    if (err.name === 'NotAllowedError') {
      error.value += 'you need to grant camera access permission'
    } else if (err.name === 'NotFoundError') {
      error.value += 'no camera on this device'
    } else if (err.name === 'NotSupportedError') {
      error.value += 'secure context required (HTTPS, localhost)'
    } else if (err.name === 'NotReadableError') {
      error.value += 'is the camera already in use?'
    } else if (err.name === 'OverconstrainedError') {
      error.value += 'installed cameras are not suitable'
    } else if (err.name === 'StreamApiNotSupportedError') {
      error.value += 'Stream API is not supported in this browser'
    } else if (err.name === 'InsecureContextError') {
      error.value +=
        'Camera access is only permitted in secure context. Use HTTPS or localhost rather than HTTP.'
    } else {
      error.value += err.message
    }
  }
  </script>

  <style scoped>
  .error {
    font-weight: bold;
    color: red;
  }
  .barcode-format-checkbox {
    margin-right: 10px;
    white-space: nowrap;
    display: inline-block;
  }
  </style>

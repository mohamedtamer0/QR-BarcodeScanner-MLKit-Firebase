# QR-BarcodeScanner-MLKit-Firebase



## Preview
<div align="center">
<br/>
  


Camera            |  Camera           |  Gallery
:-------------------------:|:-------------------------:|:-------------------------:
<img src="https://user-images.githubusercontent.com/51374446/152658394-1c7bb83e-f537-412a-ba32-966a0a03acde.gif" width="200" height="400" />  | <img src="https://user-images.githubusercontent.com/51374446/152658410-31a1f7d0-f7cc-412d-b5e0-8be83cd50a85.gif" width="200" height="400" /> | <img src="https://user-images.githubusercontent.com/51374446/152658440-bdaf346c-9d23-4087-a3cb-f3036e1cb9a0.gif" width="200" height="400" />

 </div>
  
  ```groovy
implementation 'com.google.mlkit:barcode-scanning:17.0.2'
```

  ```kotlin
  private val CAMERA_PERMISSION_CODE = 123
  private val READ_STORAGE_PERMISSION_CODE = 113
  private val WRITE_STORAGE_PERMISSION_CODE = 113

  private lateinit var cameraLauncher: ActivityResultLauncher<Intent>
  private lateinit var galleryLauncher: ActivityResultLauncher<Intent>

  private val TAG = "MyTag"
  private lateinit var inputImage: InputImage
  private lateinit var barcodeScanner: BarcodeScanner
```


  ```kotlin
cameraLauncher = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result ->
        val data = result?.data
        try {
            val photo = data?.extras?.get("data") as Bitmap
            inputImage = InputImage.fromBitmap(photo, 0)
            processQr()
        } catch (e: Exception) {
            Log.d(TAG, "onActivityResult: " + e.message)
        }
    }


    galleryLauncher = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result ->
        val data = result?.data
        inputImage = InputImage.fromFilePath(this@MainActivity, data?.data!!)
        processQr()
    }
```

  ```kotlin
binding.btnScannerBarcode.setOnClickListener {
        val options = arrayOf("camera", "gallery")
        val builder = AlertDialog.Builder(this)
        builder.setTitle("Pick a option")

        builder.setItems(options, DialogInterface.OnClickListener { _, which ->
            if (which == 0) {
                val cameraIntent = Intent(MediaStore.ACTION_IMAGE_CAPTURE)
                cameraLauncher.launch(cameraIntent)
            } else {
                val storageIntent = Intent()
                storageIntent.type = "image/*"
                storageIntent.action = Intent.ACTION_GET_CONTENT
                galleryLauncher.launch(storageIntent)
            }
        })

        builder.show()
    }
```

  ```kotlin
private fun processQr() {
  binding.ivQrCode.visibility = View.GONE
  binding.tvResult.visibility = View.VISIBLE

  barcodeScanner.process(inputImage).addOnSuccessListener {
      for (barcode: Barcode in it) {
          val valueType = barcode.valueType


          when (valueType) {
              Barcode.TYPE_WIFI -> {
                  val ssid = barcode.wifi!!.ssid
                  val password = barcode.wifi!!.password
                  val type = barcode.wifi!!.encryptionType

                  binding.tvResult.text = "ssid $ssid \n password $password \n type $type"
              }
              Barcode.TYPE_URL -> {
                  val title = barcode.url!!.title
                  val url = barcode.url!!.url
                  binding.tvResult.text = "title $title \n url $url"
              }
              Barcode.TYPE_URL -> {
                  val data = barcode.displayValue
                  binding.tvResult.text = "Result $data"
              }
          }


      }
  }.addOnFailureListener {
      Log.d(TAG, "processQr: ${it.message}")
  }
}
```

  ```kotlin
    override fun onResume() {
        super.onResume()
        checkPermission(Manifest.permission.CAMERA, CAMERA_PERMISSION_CODE)
    }
```

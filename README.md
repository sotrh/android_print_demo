# Simple demo that can print a pdf

## From MainActivity.kt

```kotlin
private fun printPdf() {
    val printManager = getSystemService(Context.PRINT_SERVICE) as PrintManager
        val jobName = "${getString(R.string.app_name)} - Printed Pdf"
        printManager.print(jobName, object : PrintDocumentAdapter() {
            override fun onWrite(pages: Array<out PageRange>?, destination: ParcelFileDescriptor?, cancellationSignal: CancellationSignal?, callback: WriteResultCallback?) {
                var input: InputStream? = null
                var output: OutputStream? = null

                try {
                    input = assets.open("learnopengl_book.pdf")
                    output = FileOutputStream(destination?.fileDescriptor)

                    val buffer = ByteArray(1024)
                    var bytesRead = 0

                    while (input?.read(buffer)?.also { bytesRead = it } ?: 0 > 0) {
                        output.write(buffer, 0, bytesRead)
                    }

                    callback?.onWriteFinished(arrayOf(PageRange.ALL_PAGES))

                } catch (fnfe: FileNotFoundException) {
                    displayMessage("The specified file does not exist")
                    fnfe.printStackTrace()
                } catch (e: Exception) {
                    displayMessage("Unable to print pdf")
                    e.printStackTrace()
                } finally {
                    try {
                        input?.close()
                        output?.close()
                    } catch (ioe: IOException) {
                        ioe.printStackTrace()
                    }
                }
            }

            override fun onLayout(oldAttributes: PrintAttributes?, newAttributes: PrintAttributes?, cancellationSignal: CancellationSignal?, callback: LayoutResultCallback?, extras: Bundle?) {
                if (cancellationSignal?.isCanceled ?: false) {
                    callback?.onLayoutCancelled()
                    return
                }

                val printDocumentInfo = PrintDocumentInfo.Builder(jobName)
                        .setContentType(PrintDocumentInfo.CONTENT_TYPE_DOCUMENT)
                        .build()

                callback?.onLayoutFinished(printDocumentInfo, true)
            }
        }, null)
}
```
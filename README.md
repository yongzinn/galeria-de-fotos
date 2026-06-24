# galeria-de-fotos
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>

package com.example.galery_photos

import android.net.Uri
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.ImageView
import androidx.recyclerview.widget.RecyclerView

class ImageAdapter(private val listaImagens: List<Uri>) :
    RecyclerView.Adapter<ImageAdapter.ImageViewHolder>() {

    class ImageViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        val imageView: ImageView = view.findViewById(R.id.ivFoto)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ImageViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_foto, parent, false)
        return ImageViewHolder(view)
    }
    override fun onBindViewHolder(holder: ImageViewHolder, position: Int) {
        val uri = listaImagens[position]
        holder.imageView.setImageURI(uri)
    }

    override fun getItemCount(): Int = listaImagens.size
}

package com.example.galery_photos

import android.Manifest
import android.content.ContentUris
import android.content.pm.PackageManager
import android.net.Uri
import android.os.Build
import android.os.Bundle
import android.provider.MediaStore
import android.widget.Toast
import androidx.activity.result.contract.ActivityResultContracts
import androidx.appcompat.app.AppCompatActivity
import androidx.core.content.ContextCompat
import androidx.recyclerview.widget.GridLayoutManager
import androidx.recyclerview.widget.RecyclerView

class MainActivity : AppCompatActivity() {

    private lateinit var rvGaleria: RecyclerView
    private val listaFotos = mutableListOf<Uri>()

    private val permissaoNecessaria = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
        Manifest.permission.READ_MEDIA_IMAGES
    } else {
        Manifest.permission.READ_EXTERNAL_STORAGE
    }

    private val solicitarPermissaoLauncher = registerForActivityResult(
        ActivityResultContracts.RequestPermission()
    ) { concedida ->
        if (concedida) {
            carregarFotos()
        } else {
            Toast.makeText(this, "Permissão negada para acessar as fotos.", Toast.LENGTH_LONG).show()
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        rvGaleria = findViewById(R.id.rvGaleria)
        rvGaleria.layoutManager = GridLayoutManager(this, 3)

        verificarPermissao()
    }

    private fun verificarPermissao() {
        when {
            ContextCompat.checkSelfPermission(this, permissaoNecessaria) == PackageManager.PERMISSION_GRANTED -> {
                carregarFotos()
            }
            else -> {
                solicitarPermissaoLauncher.launch(permissaoNecessaria)
            }
        }
    }

    private fun carregarFotos() {
        val colecao = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
            MediaStore.Images.Media.getContentUri(MediaStore.VOLUME_EXTERNAL)
        } else {
            MediaStore.Images.Media.EXTERNAL_CONTENT_URI
        }

        val projecao = arrayOf(MediaStore.Images.Media._ID)
        val ordemOrdenacao = "${MediaStore.Images.Media.DATE_ADDED} DESC"

        contentResolver.query(colecao, projecao, null, null, ordemOrdenacao)?.use { cursor ->
            val idColuna = cursor.getColumnIndexOrThrow(MediaStore.Images.Media._ID)

            listaFotos.clear() // Limpeza preventiva de fluxo
            while (cursor.moveToNext()) {
                val id = cursor.getLong(idColuna)
                val uriConteudo = ContentUris.withAppendedId(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, id)
                listaFotos.add(uriConteudo)
            }
        }
        rvGaleria.adapter = ImageAdapter(listaFotos)
    }
}

<resources>
    <string name="app_name">Galery-photos</string>
    <string name="foto_da_galeria">Foto da Galeria</string>
</resources>

<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/rvGaleria"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layoutManager="androidx.recyclerview.widget.GridLayoutManager"
        app:spanCount="3"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        tools:listitem="@layout/item_foto" />

</androidx.constraintlayout.widget.ConstraintLayout>

<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

</androidx.constraintlayout.wid

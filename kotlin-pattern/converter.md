# [Spring Boot] Converter - convert data from different types to one single type

## Background
Following example:

```kotlin

interface Convertable

data class Book (
    val title: String,
    val author: String,
) : Convertable 

data class Film(
    val title: String,
    val studio: String,
) : Convertable

data class MediaItem(
    val title: String,
    val rating: Int,
) 

```

Books and films should be converted to MediaItems to show them in a list. 
The ratings are coming from different Spring Boot services, so a `Book.toMediaItem(): MediaItem` function in the data classes does not work.

## Converters

```kotlin
interface Converter<T : Convertable> {
    
    fun convertToMediaItem(convertable: T): MediaItem
    fun canConvert(convertable: Convertable): Boolean
}

@Service
class BookConverter(
    private val bookRatingService: BookRatingService,
) : Converter<Book> {
    
    override fun convertToMediaItem(convertable: Book): MediaItem {
        return MediaItem(
            title = convertable.title,
            rating = bookRatingService.getRating(convertable)
        )
    }

    override fun canConvert(convertable: Convertable): Boolean {
        return convertable is Book
    }
}

@Service
class FilmConverter (
    private val filmRatingService: FilmRatingService,
): Converter<Film> {
    
    override fun convertToMediaItem(convertable: Film): MediaItem {
        return MediaItem(
            title = convertable.title,
            rating = filmRatingService.getRating(convertable)
        )
    }

    override fun canConvert(convertable: Convertable): Boolean {
        return convertable is Film
    }
}
```

Looks good so far, but using the converters is tricky. Let's use them in a service:

```kotlin
@Service
class MediaItemService(
    private val converters: List<Converter<in Convertable>>,
) {
    
    fun convertToMediaItem(convertable: Convertable): MediaItem {
        return converters
            .single {it.canConvert(convertable)}
            .map { it.convertToMediaItem(convertable) }
    }
}
```

Problem: Type information is lost at runtime, so the converters list is empty. \
With `converters: List<Converter<*>>,` converters are available, but the compiler can't check the types. 

## Solution

```kotlin
abstract class Converter<T : Convertable> {
    fun convertToMediaItem(convertable: Any): MediaItem? = 
        getMediaType(convertable)?.let{ return convert(mediaType) }
    protected abstract fun convert(t: T): MediaItem?
    protected abstract fun getMediaType(mediaItem: Any): T?
}

@Service
class FilmConverter (
    private val filmRatingService: FilmRatingService,
): Converter<Film> () {

    override fun convert(convertable: Film): MediaItem? {
        return MediaItem(
            title = convertable.title,
            rating = filmRatingService.getRating(convertable)
        )
    }

    override fun getMediaType(mediaItem: Any): Film? {
        return mediaItem as? Film
    }
}

// similar implementation of BookConverter
// ...

// Usage
@Service
class MediaItemService(
    private val converters: List<Converter<*>>,
) {

    fun convertToMediaItem(convertable: Convertable): MediaItem {
        return converters
            .mapNotNull { it.convertToMediaItem(convertable) }
            .single()
    }
}


```


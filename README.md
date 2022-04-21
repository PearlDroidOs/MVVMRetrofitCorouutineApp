# MVVMRetrofitCorouutineApp

Retrofit 
```
object CountriesService {

    private val BASE_URL = "https://raw.githubusercontent.com/"

    fun getCountriesService(): CountriesApi {
        return Retrofit.Builder().baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(CountriesApi::class.java)
    }
}
```

Coroutine
```
interface CountriesApi {
    @GET("DevTides/countries/master/countriesV2.json")
    suspend fun getCountries(): Response<List<Country>>
}
```
```
class ListViewModel: ViewModel() {

    val countriesService = CountriesService.getCountriesService()
    var job: Job? = null
    val exceptionHandler = CoroutineExceptionHandler { coroutineContext, throwable ->
        onError("Exception: ${throwable.localizedMessage}")
    }

    val countries = MutableLiveData<List<Country>>()
    val countryLoadError = MutableLiveData<String?>()
    val loading = MutableLiveData<Boolean>()

    fun refresh() {
        fetchCountries()
    }

    private fun fetchCountries() {
        loading.value = true

        job = CoroutineScope(Dispatchers.IO + exceptionHandler).launch {
            val response = countriesService.getCountries()

            withContext(Dispatchers.Main) {
                if (response.isSuccessful) {
                    countries.value = response.body()
                    countryLoadError.value = null
                    loading.value = false
                } else {
                    onError("Error: ${response.message()}")
                }
            }
        }
    }

    private fun onError(message: String) {
        countryLoadError.value = message
        loading.value = false
    }

    override fun onCleared() {
        super.onCleared()
        job?.cancel()
    }

}
```

<img width="376" alt="Screen Shot 2565-04-21 at 18 51 39" src="https://user-images.githubusercontent.com/62892558/164452332-2ab0f6f1-18a8-4a71-9a9f-43ea0865bd6a.png">

# Bus Scheduler App

This folder contains the source code for the Bus Scheduler app codelab.

# Introduction
The Bus Scheduler app displays a list of bus stops and arrival times. Tapping a bus stop on the first screen will display a list of all arrival times for that particular stop.

The bus stops are stored in a Room database. Schedule items are represented by the `Schedule` class and queries on the data table are made by the `ScheduleDao` class. The app includes a view model to access the `ScheduleDao` and format data to be display in a list, using `Flow` to send data to a recycler view adapter.

# Pre-requisites
* Experience with Kotlin syntax.
* Familiarity with activities, fragments, and recycler views.
* Basic knowledge of SQL databases and performing basic queries.

# Getting Started
1. Install Android Studio, if you don't already have it.
2. Download the sample.
3. Import the sample into Android Studio.
4. Build and run the sample.

# DAO (Data Access Objects)

To show all the bus stops in ascending order by arrival time. In this use case, the query just needs to get all columns and include an appropriate ORDER BY clause. The query is specified as a string passed into a @Query annotation. Define a function getAll() that returns a List of Schedule objects including the @Query annotation as shown.
```
@Query("SELECT * FROM schedule ORDER BY arrival_time ASC")
fun getAll(): List<Schedule>
```

When you want results that match the selected stop name, so you need to add a WHERE clause. You can reference Kotlin values from the query by preceding it with a colon (:) (e.g. :stopName from the function parameter). Like before, the results are ordered in ascending order by arrival time. Define a getByStopName() function that takes a String parameter called stopName and returns a List of Schedule objects, with a @Query annotation as shown.
```
@Query("SELECT * FROM schedule WHERE stop_name = :stopName ORDER BY arrival_time ASC")
fun getByStopName(stopName: String): List<Schedule>
```

# Why to use factories?
Although you've finished defining the view model, you can't just instantiate a ViewModel directly and expect everything to work. The ViewModel needs from is meant to be lifecycle aware, it should be instantiated by an object that can respond to lifecycle events. If you instantiate it directly in one of your fragments, then your fragment object will have to handle everything all the memory management, which is beyond the scope of what your app's code should do. Instead, you can create a class, called a factory, that will instantiate view model objects for you.
```
override fun <T : ViewModel> create(modelClass: Class<T>): T {
       if (modelClass.isAssignableFrom(BusScheduleViewModel::class.java)) {
           @Suppress("UNCHECKED_CAST")
           return BusScheduleViewModel(scheduleDao) as T
       }
       throw IllegalArgumentException("Unknown ViewModel class")
   }
```

# Setting the database
When using an AppDatabase class, you want to ensure that only one instance of the database exists to prevent race conditions or other potential issues. The instance is stored in the companion object, and you'll also need a method that either returns the existing instance, or creates the database for the first time. We use the @Volatile annotation on the INSTANCE variable to avoid potential bugs.

```
@Database(entities = arrayOf(Schedule::class), version = 1)
abstract class AppDataBase : RoomDatabase() {
    abstract fun scheduleDao(): ScheduleDao

    companion object {
        @Volatile
        private var INSTANCE: AppDataBase? = null

        fun getDatabase(context: Context): AppDataBase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context,
                    AppDataBase::class.java,
                    "app_database"
                ).createFromAsset("database/bus_schedule.db")
                    .build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

Note: The version number is incremented each time you make a schema change. The app checks this version with the one in the database to determine if and how a migration should be performed.

# Application class
We'll need to provide a custom subclass of the Application class, and create a lazy property that will hold the result of getDatabase().
```
class BusScheduleApplication:Application() {
    val database:AppDataBase by lazy { AppDataBase.getDatabase(this) }
}
```
To make sure that BusScheduleApplication class is used (instead of the default base class Application), we need to make a small change to the manifest. In AndroidMainifest.xml, set the android:name to com.example.busschedule.BusScheduleApplication.
```<application
    android:name="com.example.busschedule.BusScheduleApplication"
    ...
```

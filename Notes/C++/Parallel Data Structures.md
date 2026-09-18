* `std::thread`
* `std::mutex`
	* `lock()`
	* `try_lock()`
	* `unlock()` [undefined behavior if current thread does not own lock]
	* Other std lib
		* `std::recursive_mutex`
		* `std::timed_mutex`
		* `std::recursive_timed_mutex`
		* `std::shared_mutex`
		* `std::stared_timed_mutex`
* lock guards 
	* RAII wrapper around mutexes
		* constructor locks
		* destructor unlocks
	* `std::scoped_lock` (C++ 17)
		*  `std::scoped_lock lock(mutex_a, mutex_b);`
		* useful for avoiding deadlocks - all mutex parameters locked in the same order
	* `std::lock_guard` (pre C++17)
		* `std::lock_guard<std::mutex> lock(mutex_a);`
	* `std::unique_lock`
		* owns a mutex
	* `std::shared_lock`
		* `std::shared_lock lock(shared_mutex_a);`
* `std::atomic`
	* without anything else no data race
	* implicit read before write synchronized
	* good for primitive types
* `std::conditional_variable`
* `std::counting_semaphore`
* `std::latch`
* `std::barrier`
* 
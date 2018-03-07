# BM #
Minimalist benchmarking library.

## Abstractions ##

#### `bm::record<type>` #####
Simple struct containing a vector. 
Provides functionality to compute the mean, variance and standard deviation. Exports to csv.

```cpp
template<typename type = double>
struct record
{
  type mean              () {...}
  type variance          () {...}
  type standard_deviation() {...}

  void to_csv 			 (const std::string& filepath) {...}
  
  std::vector<type> values;
}
```

#### `bm::session<type>` ####
Simple struct containing a flat map of records, indexed by a name. 
Exports to csv. 
Only used for macro-benchmarking.

```cpp
template<typename type = double>
struct session
{
  void to_csv(const std::string& filepath, const bool include_name = true) {...}
  
  std::vector<std::pair<std::string, record<type>>> records;
}
```

#### `bm::recorder<type, period>` ####
Helper class providing a single public method accepting a name and a function. 
The function is run once, and its duration is appended to an internally managed session indexed by name. 
Only used for macro-benchmarking.

```cpp
template <typename type = double, typename period = std::milli>
class recorder
{
public:
  void record(const std::string& name, const std::function<void()>& function) const {...}
}

```

#### `bm::run<type, period>` ####
The entry function which runs a benchmark and creates records or sessions. Provides two overrides for micro- and macro-benchmarking.

```cpp
template<typename type = double, typename period = std::milli>
record<type> run(const std::size_t iterations, const std::function<void()>& function) {...}

template<typename type = double, typename period = std::milli>
session<type> run(const std::size_t iterations, const std::function<void(recorder<type, period>&)>& function) {...}
```

## Example Usage ##

```cpp
#include <algorithm>
#include <cstddef>
#include <vector>

#include <bm/bm.hpp>

int main(int argc, char** argv)
{
  std::vector<std::size_t> vector(100000);

  // Micro-benchmarking.
  auto record = bm::run<float, std::milli>(100, [&] ()
  {
    std::iota(vector.begin(), vector.end(), 0);
  });
  auto mean = record.mean();
  auto 	 = record.mean();
  auto mean = record.mean();
  record.to_csv("iota_100000_values_100_iterations.csv");

  // Macro-benchmarking.
  auto session = bm::run<float, std::milli>(100, [&vector] (bm::recorder<float, std::milli>& recorder)
  {
    recorder.record("iota", [&vector] ()
    {
      std::iota(vector.begin(), vector.end(), 0);
    });
    recorder.record("generate", [&vector] ()
    {
      std::generate(vector.begin(), vector.end(), std::rand);
    });
  });
  session.to_csv("iota_then_generate_100000_values_100_iterations.csv", true);
}
```
title: C++11新特性概览
date: 2017-01-09 22:38:35
tags:
 - C++
---

今天买的《C++ Primer 第五版》到了，这一版本一个比较好的地方是。在开始的目录里面列出来了全书中涉及到的C++11新特性的地方，标明了页码，可以直接到对应的页面去看新特性的东西。于是我对照书上的例子，写了一些简单的示例，用来大概的了解C++11的新特性，总结在这里，以后可以查查。
<!--more-->

```cpp
#include <iostream>
#include <cstdlib>
#include <vector>
#include <array>
#include <iterator>
#include <functional>

using namespace std;

constexpr int size(){return 100;} // constexpr function, compiler will convert it to inline function 

void func(initializer_list<int> li) {
	cout << "func with " << li.size() << "elements" << endl;
}	

//
vector<int> func2() {
	return {1,2,3};
}


int main() {
	long long ll = 64; //long long: 64 bit at least

	//2. list initialization
	int a = {0};
	int bb{0};
	vector<int> v2 = {1,2,3};


	//3. nullptr, a new literal
	int *p = nullptr;
	//equals to 
	int *p2 = 0;
	int *p3 = NULL;// must include <cstdlib> first

	//4. constexpr
	constexpr int ci = 30;
	constexpr float cf = ci * 0.2;
	constexpr int sz = size(); // it's ok to initialize a constexpr value using constexpr function

	//5. alias declaration
	using my_int = int;
	my_int i = 10;

	using vi = vector<int>;
	vi v = {1,2,3,5};

	using it = vector<int>::iterator;
	for (it it_i = v.begin(); it_i != v.end(); ++it_i) {
		cout << *it_i << endl;
	}


	//6. auto
	auto aa = 3; // compiler will infer that a's type is int

	//7.decltype
	decltype(i) id1 = 0; // id1 has the same type with i
	decltype((i)) id2 = id1; // NOTE: (var) returns reference! so id2 is a reference to id1

	//7. in-class initializer 
	/*
	struct my_struct {
		int i = 0;
		int b = 2;
		string s;
	};
	*/

	//8. range for
	string hello = "hello, world!";
	for (auto c : hello) {
		cout << c << endl;
	}

	//9. vector of vector, no space anymore!
	//before:
	vector<vector<int> > vv1; 
	//now:
	vector<vector<int>> vv2;

	//10. cbegin, cend, return const_iterator don't care if container is const
	vector<int> vi10 = {4,4,54};
	const vector<int> cvi10 = {4,4,54};
	auto it10 = vi10.cbegin(); // it10: const_iterator type
	auto cit10 = cvi10.cbegin(); // cit10: const_iterator type

	//11. begin and end of pointer, included in <iterator>
	int ia[] = {1,3,4};
	int *b = begin(ia);
	int *e = end(ia);
	
	//12. initializer_list
	func({1,2,3,4});

	//13. trailing return type
	/*
	auto func(int i) -> int(*)[10];
	*/

	//14. =default: don't replace default initializer function
	/*
	class A {
		public:
		A() = default;
	};
	*/

	//15. delegating constructor
	/*
	class A {
	  public:
		A(int i, float f, double d):ii(i), ff(f), dd(d) {}
		A(int i, float f): A(i, f, 0) {}
		A(int i): A(i, 0, 0) {}

	  private:
		int ii;
		float ff;
		double dd;
	};
	*/
	
	//16. array 
	array<int, 3> a1 = {{1,2,3}};
	auto aa1 = a1.begin();
	
	//17.forward_list

	//18. emplace

	//19. shrink_to_fit

	//20. to_string
	int i20 = 233;
	string s20 = to_string(i20);

	//21. lambda 
	auto f211 = [] () {return 23;};
	auto f212 = [i20]() {return i20 > 20;};
	auto f213 = [i20]() -> int {if(i20> 20) return 1; else return 0;};

	//22. bind, included in <functional>
	/*
	int f(int a, int b) {
		return a>b ? a : b;
	}

	auto g = bind(f, _1, 10);
	*/
	
	//23. unordered associative container, managed by hash
	//including unordered_map, unordered_set, unordered_multimap, unordered_multiset

	//24. smart pointer,including shared_ptr, unique_ptr, weak_ptr

	//25. rvalue reference
	int i25 = 42; 
	int && rr = i25*42;

	//ERROR: variable is lvalue
	//int &&rr2 = rr;

	//26. std::move
	int &&rr3 = std::move(rr);

	//27. noexcept,use for moving copy or moving operator

	//28. reference qualifier
	/*
	class A{
	  public:
	  	A sorted() &&; //rvalue reference
		A sorted() &; //lvalue reference
	};
	*/

	//29. function template, included in <functional>
	//function<int(int, int)> f1 = add; //add is a function declared before

	//30. explicit conversion operator
	/*
	class A {
		explicit operator int() const {return val;};
	};
	*/
	
	//31. final to prevent inherit
	class Last final {}; // Last can't be a basic class

	//32.override: override virtual functions in parents class

	//33. tuple
	tuple<int, int, int> tt{1,2,4};
	auto tt1 = get<0>(tt);
	auto tt2 = get<1>(tt);
	auto tt3 = get<2>(tt);


	return 0;
}
```

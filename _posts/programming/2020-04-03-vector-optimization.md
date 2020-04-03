---
layout: post
title:  "Vector optimization in a game engine"
date:   2020-04-03
desc: "My experience on optimizing vectors in a game engine."
keywords: "Canas,Simon,gh-pages,website,blog,engine,vector,optimization"
categories: [Programming]
tags: [Programming,Engine]
icon: icon-cplusplus 
---

Vectors are an integral part of any game engine and are used in many of its aspects.
They are necessary to represent the position, rotation and scale of objects, physical interactions like contacts and collisions, and in some instances to depict matrices rows or columns (depending on who you are talking to).

It is needless to say that since this component is so often used in any game engine, it needs to be as fast as possible in order to squeeze every bit of performance out of the engine.

<div class="navy-line-no-margin"></div>

During my second year of study, me and my classmates have all been tasked to take part in the creation of a game engine, with our teacher as the lead of the project.
Once every two weeks we were given a new task and every week we would discuss each student's advancement and our teacher would give pointers on how to improve our current code.

I have been given multiple tasks over the course of this project, but I think the most interesting one was when I worked with vectors, which is what I'm going to present in this post.

<div class="navy-line-no-margin"></div>
# First implementation

I needed to create a class for each of the 3 dimensions used in game engines, being Vec2, Vec3 and Vec4 with the only thing 

Here is an example of the 3D vector implementation:

```cpp
template<typename T>
class Vec3
{
public:
    union
    {
        struct
        {
            T x;
            T y;
            T z;
        };
        T coord[3];
    };

    const static Vec3 zero;
    const static Vec3 one;
    const static Vec3 up;
    const static Vec3 down;
    const static Vec3 left;
    const static Vec3 right;
    const static Vec3 forward;
    const static Vec3 back;
```

As you can see, the implementation is pretty simple: a **struct** that contains the vector coordinates and an array to, for example, iterate through the vector coordinates. All of this in a template class, to be able to handle multiple types. There are also some static variables to make some usual vectors readily accessible.

After that, most of the work was to create the multiple mathematical methods that would be used in the engine, for example:
```cpp
static T Dot(const Vec3<T> v1, const Vec3<T> v2)
{
    return v1.x * v2.x + v1.y * v2.y + v1.z * v2.z;
}

static Vec3<T> Cross(const Vec3<T> v1, const Vec3<T> v2)
{
    return Vec3<T>(v1.y * v2.z - v1.z * v2.y,
                   v1.z * v2.x - v1.x * v2.z,
                   v1.x * v2.y - v1.y * v2.x);
}

template<typename ReturnT = float>
ReturnT Magnitude() const
{
    return std::sqrt(SquareMagnitude());
}

static Vec3<T> Lerp(const Vec3<T>& v1, const Vec3<T>& v2, T t)
{
    return v1 + (v2 - v1) * t;
}
```

And that is basically it, vectors are not that complicated after all (it would that much more painful to make a physics engine otherwise).
However, I kept thinking "That's it?" after finishing all this, there had to be some sort of optimization to apply here.

<div class="navy-line-no-margin"></div>
# The problem with vectors

When I presented my work on the first week, apart from other minor mishaps on my part, my code did not really have any major optimization flaw (there are not many ways to implement vectors ultimately). My teacher did, however, pointed me towards something we had learned recently: **Intrinsics commands**.

Vectors have one small flaw in every implementation: a singular vector coordinates are aligned, but when operations with multiple vectors are at play, the values are not aligned in the fastest way possible.

For example, if we look at those registry entries:

<style>
table {
  width: 25%;
}
</style>
<table> 
	<tbody> 
		<tr> 
			<th>rdi</th>
			<td>y1</td>
			<td>x1</td>
		</tr> 
		<tr> 
			<th>rsi</th>
			<td class="grayout"></td>
			<td>z1</td>
		</tr> 
		<tr> 
			<th>rdx</th>
			<td>y2</td>
			<td>x2</td>
		</tr> 
		<tr> 
			<th>rsi</th>
			<td class="grayout"></td>
			<td>z2</td>
		</tr> 
	</tbody>
</table>

As you can see, the x, y and z coordinates of the two vectors are not aligned together, and there also some wasted space in the registries.

The goal would be to obtain this:

<table> 
	<tbody> 
		<tr> 
			<th>rdi</th>
			<td>x2</td>
			<td>x1</td>
		</tr> 
		<tr> 
			<th>rsi</th>
			<td>y2</td>
			<td>y1</td>
		</tr> 
		<tr> 
			<th>rdx</th>
			<td>z2</td>
			<td>z1</td>
		</tr> 
		<tr> 
			<th>rsi</th>
			<td class="grayout"></td>
			<td class="grayout"></td>
		</tr> 
	</tbody>
</table>

Space is used more efficiently and every coordinates are correctly aligned together.

After some more pointers given by my teacher, it became clear what more I could do...

<div class="navy-line-no-margin"></div>
# N-Vectors

The goal was to be able to align multiple vectors coordinates in order to use intrinsics commands on those. From that, came the NVec classes:

```cpp
template<typename T, int N>
class alignas(N * sizeof(T)) NVec3
{
public:
    std::array<T, N> xs;
    std::array<T, N> ys;
    std::array<T, N> zs;
```

This class stores each vector coordinates in a separate array that is aligned on the size of the type **(T)** times the number of vectors given **(N)**.

And now that we have our values aligned, it is time for some intrinsics!

<div class="navy-line-no-margin"></div>
# Intrinsics

Due to how alignas works, we can only use NVec with a size in a power of 2. In the current implementations, it only handles **FourVec**s and **EightVec**s. Each function has an intrinsic and non-intrinsic version. For example, for the dot product:

```cpp
static std::array<T, N> Dot(NVec3<T, N> v1, NVec3<T, N> v2)
{
    std::array<T, N> result;
    for (int i = 0; i < N; i++)
    {
        result[i] = v1.xs[i] * v2.xs[i] +
                    v1.ys[i] * v2.ys[i] +
                    v1.zs[i] * v2.zs[i];
    }
    return result;
}

template <>
inline std::array<float, 4> FourVec3f::DotIntrinsics(FourVec3f v1, FourVec3f v2)
{
    alignas(4 * sizeof(float))
    std::array<float, 4> result;
    auto x1 = _mm_load_ps(v1.xs.data());
    auto y1 = _mm_load_ps(v1.ys.data());
    auto z1 = _mm_load_ps(v1.zs.data());

    auto x2 = _mm_load_ps(v2.xs.data());
    auto y2 = _mm_load_ps(v2.ys.data());
    auto z2 = _mm_load_ps(v2.zs.data());

    x1 = _mm_mul_ps(x1, x2);
    y1 = _mm_mul_ps(y1, y2);
    z1 = _mm_mul_ps(z1, z2);

    x1 = _mm_add_ps(x1, y1);
    x1 = _mm_add_ps(x1, z1);
    _mm_store_ps(result.data(), x1);
    return result;
}
```

The first method is pretty straightforward: we make a for loop to calculate the dot between each element of **v1** and **v2**.

For the intrinsic method, we start by creating an array to store our result in and align it on the correct amount of bytes, because otherwise it might crash on some compilers. We then load the coordinates using **_mm_load_ps**. Next, we store the results of **_mm_mul_ps** and **_mm_add_ps** directly in the first argument because we won't be needing the value for later. Finally, **x1** is stored into the result array with **_mm_store_ps** which is then returned by the method.

Funnily enough, I initially encountered errors when doing the same methods for **EightVec**s, when what I just needed what to change the **_mm** prefix by **_mm256**... simple as that.

For every size that is not 4 or 8, the regular dot should be called, otherwise it will throw an error.

Now, with everything set up, it is time to do some benchmarking to see whether the intrinsics are really faster than the regular approach or not:

```bash
-----------------------------------------------------------------------
Benchmark                             Time             CPU   Iterations
-----------------------------------------------------------------------
BM_SquareMagnitudeSimple          0.587 ns        0.453 ns   1000000000
BM_SquareMagnitude                 24.7 ns         21.3 ns     37333333
BM_SquareMagnitudeIntrinsics      0.467 ns        0.422 ns   1000000000
BM_MagnitudeSimple                0.368 ns        0.359 ns   1000000000
BM_Magnitude                       22.7 ns         22.2 ns     34461538
BM_MagnitudeIntrinsics            0.301 ns        0.297 ns   1000000000
BM_DotSimple                       8.42 ns         8.54 ns     89600000
BM_Dot                             24.5 ns         24.6 ns     28000000
BM_DotIntrinsics                  0.376 ns        0.359 ns   1000000000
BM_ReflectSimple                   14.2 ns         13.7 ns     56000000
BM_Reflect                          134 ns          136 ns      4480000
BM_ReflectIntrinsics              0.391 ns        0.375 ns   1000000000
```

I used a NVec of size 8 for this benchmark.
In order, we have: without NVec, using Nvec non-intrinsic methods and using Nvec intrinsic methods.

And... Hooray! The intrinsic methods is indeed faster, and in some instances by a significant amount! This also make some complex methods like **Reflect()** to be just as fast as a simple one like **Dot()** which will probably be very useful for physics collisions.
However, the Nvec non-intrinsic methods are way slower that the regular way... probably the compiler doing some fancy behind the scene optimization, making them quite useless.

<div class="navy-line-no-margin"></div>
# Conclusion

This exercise was a great to experiment with intrinsics and to show the kind how power they possess for optimizing a game engine. It also helped me familiarize much more with the concept of data alignement and registries.
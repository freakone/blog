---
layout: post
title: STM USB HAL library HardFault bug
---

I decided to use the HAL USB library to handle communication with a PC (CDC class implementation) in the F303 processor. I ran into a strange issue: after a PC reboot device was freezing every time.

# Investigation
A quick look into debugger revealed that `HardFault` exception is raised. After stack trace investigation it turned out that the scenario of the fail was pretty simple.

1. After close command in OS, USB detaches all devices.
2. Interrupt from USB bus is fired and `USBD_CDC_DeInit` callback is executed.
3. Library is closing bus and freeing all resources.
4. **Error!**

# Library bug

Program fails at freeing memory (``line 585, usbd_cdc.c``):

{% highlight c %}
USBD_free(pdev->pClassData);
{% endhighlight %}

When you trace this function it leads to simple `free()` instruction, nothing fancy. But when you look up a little bit you will find a function which should allocate memory in a way that could be released through `free()` function...

{% highlight c %}
/**
  * @brief  static single allocation.
  * @param  size: size of allocated memory
  * @retval None
  */
void *USBD_static_malloc(uint32_t size)
{
  static uint32_t mem[(sizeof(USBD_CDC_HandleTypeDef)/4)+1];//On 32-bit boundary
  return mem;
}

/**
  * @brief  Dummy memory free
  * @param  *p pointer to allocated  memory address
  * @retval None
  */
void USBD_static_free(void *p)
{
  free(p);
}
{% endhighlight %}

... but it is not. It just returns a pointer to a static variable, so there is nothing to be `free()` at the end!
Quick look at [free function reference](http://www.cplusplus.com/reference/cstdlib/free/):

```
Deallocate memory block

A block of memory previously allocated by a call to malloc, calloc or realloc is deallocated, making it available again for further allocations.

If ptr does not point to a block of memory allocated with the above functions, it causes undefined behavior.
```

# Quick fix

Just comment out `free()` function.

{% highlight c %}
void USBD_static_free(void *p)
{
  //free(p);
}
{% endhighlight %}


# HAL version

This bug is present in file version presented below.

{% highlight c %}
  * @file    usbd_cdc.c
  * @author  MCD Application Team
  * @version V2.4.2
  * @date    11-December-2015
{% endhighlight %}

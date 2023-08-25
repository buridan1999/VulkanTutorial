## About

Этот гайд научит вас основам использования [Vulkan](https://www.khronos.org/vulkan/)
API. Vulkan это новые API, созданное [Khronos group](https://www.khronos.org/)
(известные по OpenGL), которое предоставляет более удобную абстракцию над современными 
видеокартами. Этот новый интерфейс позволяет вам лучше описать то, что ваше ПО 
пытается осуществить, что способствует лучшей производительности и сокращает ситуации, 
когда драйвер ведёт себя нестандартным образом, в сравнении с текущими API, как 
[OpenGL](https://ru.wikipedia.org/wiki/OpenGL) и [Direct3D](https://en.wikipedia.org/wiki/Direct3D). 
Vulkan основан на похожих идеях, которые применялись в [Direct3D 12](https://en.wikipedia.org/wiki/Direct3D#Direct3D_12)
и [Metal](https://en.wikipedia.org/wiki/Metal_(API)), но у Vulkan есть преимущество - 
кроссплатформенность, которое позволяет вам разрабатывать для Windows, Linux и
Android одновременно.

Однако, получая данные преимущества, теперь вам нужно использовать более сложное
API. Каждая деталь относящаяся к графическому API должна быть настроена с нуля
вашим приложением, включая создание frame buffer и управление памятью для таких
объектов как буфферы и текстуры. Графический драйвер будет делать гораздо меньше 
проверок на корректность поступающих команд, что означает, то что вам придётся 
осуществлять данную работу самому, чтобы быть уверенным в корректном поведении ПО.

Вывод: Vulkan не для всех. Данное API нацелено на программистов, которым нужна
высокая производительность в компьютерной графике, и которые готовы потратить 
на это время. Если вы  интересуетесь разработкой игр больше, чем компьютерной 
графикой, тогда вам стоит обратить своё внимание на OpenGL и Direct3D, которые 
точно не станут устаревшими на фоне Vulkan в близжайшее время. Другая
альтернатива, использовать игровой двидок, как [Unreal Engine](https://en.wikipedia.org/wiki/Unreal_Engine#Unreal_Engine_4)
или [Unity](https://en.wikipedia.org/wiki/Unity_(game_engine)), которые способны
использовать Vulkan, предоставляя более высокоуровневое API.

Разобравшись с этим, давайте рассмотрим некоторые предварительные условия 
для выполнения этого руководства:

* Графический адаптер и драйвер совместимые с Vulkan ([NVIDIA](https://developer.nvidia.com/vulkan-driver), [AMD](http://www.amd.com/en-us/innovations/software-technologies/technologies-gaming/vulkan), [Intel](https://software.intel.com/en-us/blogs/2016/03/14/new-intel-vulkan-beta-1540204404-graphics-driver-for-windows-78110-1540), [Apple Silicon (Or the Apple M1)](https://www.phoronix.com/scan.php?page=news_item&px=Apple-Silicon-Vulkan-MoltenVK))
* Опыт использования C++ (понимание RAII, initializer lists)
* Компилятор с поддержкой возможностей стандарта C++17 (Visual Studio 2017+, GCC 7+, Or Clang 5+)
* Опыт работы с 3D графикой

Данный гайд не предполагает понимание концептов OpenGL или Direct3D, но требует
от вас понимания основ компьютерной 3D графики. Здесь не будут объясняться такие
темы из математики, как проекция перспективы, например. Для введения в компьютерную 
графику, смотрите [данную электронную книгу](https://paroj.github.io/gltut/).
Вот ещё несколько хороших материалов по данной теме:

* [Ray tracing за выходные](https://github.com/RayTracing/raytracing.github.io)
* [PBR - Книга по "Физически корректному рендерингу"](http://www.pbr-book.org/)
* Vulkan используют в настоящих игровых движках с открым исходным кодом [Quake](https://github.com/Novum/vkQuake) и [DOOM 3](https://github.com/DustinHLand/vkDOOM3)

Вы можете использовать C вместо C++, если хотите, но вам также придется использовать 
библиотеку линейной алгебры, и вы будете предоставлены сами себе в плане структурирования кода.
Мы будем использовать возможности C++, такие как классы и RAII, для организации логики и
времени жизни объектов. Также есть две альтернативные версии этого руководства, доступные для разработчиков Rust: [Vulkano based](https://github.com/bwasty/vulkan-tutorial-rs), [Vulkanalia based](https://kylemayes.github.io/vulkanalia).

Чтобы разработчикам, использующим другие языки программирования, было проще ориентироваться, и чтобы получить некоторый опыт работы с базовым API, мы будем использовать оригинальный C API для работы с Vulkan. Однако, если вы используете C++, вы можете предпочесть использовать более новую обёртку для C++ [Vulkan-Hpp](https://github.com/KhronosGroup/Vulkan-Hpp ), которая абстрагируют часть грязной работы и помогает предотвратить определенные классы ошибок.

## E-book

If you prefer to read this tutorial as an e-book, then you can download an EPUB
or PDF version here:

* [EPUB](https://vulkan-tutorial.com/resources/vulkan_tutorial_en.epub)
* [PDF](https://vulkan-tutorial.com/resources/vulkan_tutorial_en.pdf)

## Tutorial structure

We'll start with an overview of how Vulkan works and the work we'll have to do
to get the first triangle on the screen. The purpose of all the smaller steps
will make more sense after you've understood their basic role in the whole
picture. Next, we'll set up the development environment with the [Vulkan SDK](https://lunarg.com/vulkan-sdk/),
the [GLM library](http://glm.g-truc.net/) for linear algebra operations and
[GLFW](http://www.glfw.org/) for window creation. The tutorial will cover how
to set these up on Windows with Visual Studio, and on Ubuntu Linux with GCC.

After that we'll implement all of the basic components of a Vulkan program that
are necessary to render your first triangle. Each chapter will follow roughly
the following structure:

* Introduce a new concept and its purpose
* Use all of the relevant API calls to integrate it into your program
* Abstract parts of it into helper functions

Although each chapter is written as a follow-up on the previous one, it is also
possible to read the chapters as standalone articles introducing a certain
Vulkan feature. That means that the site is also useful as a reference. All of
the Vulkan functions and types are linked to the specification, so you can click
them to learn more. Vulkan is a very new API, so there may be some shortcomings
in the specification itself. You are encouraged to submit feedback to
[this Khronos repository](https://github.com/KhronosGroup/Vulkan-Docs).

As mentioned before, the Vulkan API has a rather verbose API with many
parameters to give you maximum control over the graphics hardware. This causes
basic operations like creating a texture to take a lot of steps that have to be
repeated every time. Therefore we'll be creating our own collection of helper
functions throughout the tutorial.

Every chapter will also conclude with a link to the full code listing up to that
point. You can refer to it if you have any doubts about the structure of the
code, or if you're dealing with a bug and want to compare. All of the code files
have been tested on graphics cards from multiple vendors to verify correctness.
Each chapter also has a comment section at the end where you can ask any
questions that are relevant to the specific subject matter. Please specify your
platform, driver version, source code, expected behavior and actual behavior to
help us help you.

This tutorial is intended to be a community effort. Vulkan is still a very new
API and best practices have not really been established yet. If you have any
type of feedback on the tutorial and site itself, then please don't hesitate to
submit an issue or pull request to the [GitHub repository](https://github.com/Overv/VulkanTutorial).
You can *watch* the repository to be notified of updates to the tutorial.

After you've gone through the ritual of drawing your very first Vulkan powered
triangle onscreen, we'll start expanding the program to include linear
transformations, textures and 3D models.

If you've played with graphics APIs before, then you'll know that there can be a
lot of steps until the first geometry shows up on the screen. There are many of
these initial steps in Vulkan, but you'll see that each of the individual steps
is easy to understand and does not feel redundant. It's also important to keep
in mind that once you have that boring looking triangle, drawing fully textured
3D models does not take that much extra work, and each step beyond that point is
much more rewarding.

If you encounter any problems while following the tutorial, then first check the
FAQ to see if your problem and its solution is already listed there. If you are
still stuck after that, then feel free to ask for help in the comment section of
the closest related chapter.

Ready to dive into the future of high performance graphics APIs? [Let's go!](!en/Overview)

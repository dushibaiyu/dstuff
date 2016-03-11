Some little hopefully helpful D modules placed in the public domain.

### trackallocs.d ###

  A tiny hack to install a GC proxy and track all allocations, gathering some 
  stats and logging to given File (or stdout, by default).

  Use it like this:
  
    void myCode() {
      auto tracker = allocsTracker();
      ... // do some work
          // while `tracker` is alive all allocations will be shown
    }

  Alternatively use pair of functions 
    startTrackingAllocs() and stopTrackingAllocs().


### gcarena.d ###

  A tiny hack to redirect all GC allocations to a fixed size arena.

  Say, you've got an operation that actively allocates in GC'ed heap but
  after it's complete you don't need anything it allocated. And you need
  to do it in a loop many times, potentially allocating lots of memory.
  Just add one line in the beginning of loop's scope and each iteration
  will reuse the same fixed size buffer. 
  Like this:

    foreach(fname; files) {
      auto ar = useCleanArena();     // (1)    
      auto img = readPng(fname).getAsTrueColorImage();
      process(img);
      // (2)
    }
  
  Between points (1) and (2) all GC allocations will happen inside 
  an arena which will be reused on each iteration. No GC will happen,
  no garbage accumulated.

  If you need some data created inbetween, you can temporarily pause
  using the arena and allocate something on main GC heap:

    void testArena() {
        auto ar = useCleanArena();
        auto s = new ubyte[100]; // allocated in arena
        {
            auto pause = pauseArena();  // in this scope it's not used
            auto t = new ubyte[200];    // allocated in main GC heap
        }
        auto v = new ubyte[300];   // allocated in arena again
        writeln("hi");            
        // end of scope, stop using arena 
    }

  You can set size for arena by calling setArenaSize() before its first use.
  Default size is 64 MB.


### letassign.d ###

  Destructuring let assignment similar to those found in ML, Haskell etc.
  Works like this:

    int x, y, z, age;
    string name;

    let (name, age) = getTuple();           // tuple
    let (x,y,z) = argv[1..4].map!(to!int);  // lazy range
    let (x,y,z) = [1,2,3];                  // array

    SomeStruct s;
    let (s.a, s.b) = tuple(3, "piggies");
  
  where

    auto getTuple() { return tuple("Bob", 42); }  // just an example

  let (...) = ... requires equal number of elements on both sides of =
  for tuples (checked statically) and arrays (checked at runtime),
  and enough elements in an input range. If there are not enough elements
  it throws an Exception.
  
  To use just available number of elements and keep some variables 
  on the left unchanged, use 

  let (...)[] = ...


    let (x,y,z)[] = argv[1..3].map!(to!int);  // lazy range
    let (x,y,z)[] = [1,2];                    // array

  To avoid separate type declarations and work with immutables, there is
  another function `into`:


    int x = getTuple().into( (string name, int age) => age + name.length );
    assert(x==45);
 
    calcScore(prg).into((int score, bool ok) => writeln("score=", score, " ok=", ok));


  It just feeds data from a tuple as arguments to a given function.


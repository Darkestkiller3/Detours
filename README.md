# Detours
A detours library for API hooking in C#
A Library for simple hooking using C#.
Takes little to no time to setup and once setup you will be hooking like a pro in no time. 
This makes hooking simpler and easier then EasyHook or others.
First make a class to manage your hooks!
```cs

    internal class Hooks
    {
        private static readonly Detours.HookManager manager = new Detours.HookManager();

        public static Detours.HookManager Manager
        {
            get { return manager; }
        }

        [HandleProcessCorruptedStateExceptions]
        public static void FindFunctions()
        {
            try
            {
                //Here you define what functions sections to load hooks for.
                FunctionInfo.MapFunctions.AddRange(DetouredFunctions.FunctionInfo);
                foreach (var function in FunctionInfo.MapFunctions)
                {
                    function.SetTarget(
                        Marshal.GetDelegateForFunctionPointer(
                            function.Address, function.TargetType));
                    Log.LogFunctionAddress(
                        function.Name, Marshal.GetFunctionPointerForDelegate(function.Target));
                }
            }
            catch (Exception e) { Console.WriteLine(e); }
        }

        [HandleProcessCorruptedStateExceptions]
        public static void Set()
        {
            try
            {
                foreach (var function in FunctionInfo.MapFunctions)
                {
                    manager.Add(function.Target, function.Detour, function.Name);
                }

                manager.InstallAll();
            }
            catch (Exception e) { Console.WriteLine(e); }
        }
    }
}
```

Now we can actually start making some hooks!
```cs
    internal class DetouredFunctions
    {
        private static ushort DetouredFunction(int Parm)
        {
                Hooks.Manager["Game::Function"].CallOriginal(new object[] { Parm });
        }
        #region Hooks

        public static readonly DFunction Function = DetouredFunction;


        public static readonly FunctionInfo[] FunctionInfo = new[]
        {
						//Memory address goes where 0x00000000
            new FunctionInfo((IntPtr)0x00000000,"Game::Function",Function,_Function,typeof(DFunction)),
        };

        [UnmanagedFunctionPointer(CallingConvention.StdCall)]
        public delegate ushort DFunction(int nClassType);

        public static DFunction _Function { get; set; }
	}
```

Last but not least you can enable all your hooks by calling
```cs

                //Find Functions to hook
                Hooks.FindFunctions();
                //Apply the hooks
                Hooks.Set();
```

This has been a very helpful library for me just making this readme so every one else can Enjoy!
There is one known bug with this library I'm currently thinking of a work around.
Any help is appreciated.
Hooking any function with overlapping calls will cause bugs and issues because of constant rewrite of the original address.
If you overwrite the original address then call it before its rewriten back it will errors and other stuff.

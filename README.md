# D3D9Hook.NET
a semi d3d9 hooking library based on CSharp.NET

using System;
using System.Collections.Generic;
using System.Linq;
using System.Runtime.InteropServices;
using System.Text;
using System.Threading.Tasks;
using d3d9hook;
namespace d3d9hook
{
    class Program
    {
        [DllImport("kernel32.dll")]
        static extern void WriteProcessMemory(IntPtr handle,long addressakabase,byte[] writtens,int length,int rt);
        [DllImport("kernel32.dll")]
        static extern void ReadProcessMemory(IntPtr handleshit, long address, byte[] patterns, int null_ptr,int tr);
        [DllImport("kernel32.dll")]
        static extern IntPtr OpenProcess(int page_size, bool cracken_lope, int pid);
        static bool x64 = false;
        static bool x86 = false;
        static int cout(string val,int req)
        {
            Console.WriteLine(val);
            if (req == 1)
            {
                Console.ReadKey();
            }
            else
            {

            }
           
            return 0;
        }
        static long IDirectx9Device = 0x5880; //the address will be null until we got the right pattern.
        static long IDirectx9DeviceReset = 0x12D58;
        static long real_address = 0;
        [DllImport("kernel32.dll")]
        private static extern bool VirtualProtectEx(IntPtr hProcess, long lpAddress, UIntPtr dwSize, uint flNewProtect, out uint lpflOldProtect);
        public static string ByteArrayToString(byte[] ba)
        {
            StringBuilder hex = new StringBuilder(ba.Length * 2);
            foreach (byte b in ba)
                hex.AppendFormat("{0:X2} ", b);
            return hex.ToString();
        }
        static void Implement_d3d_device(long directx9device,string pname)
        {
            System.Diagnostics.Process process_d3d = System.Diagnostics.Process.GetProcessesByName(pname)[0];
            
            IntPtr d3dProcessHandle = OpenProcess(0x0FFFFF, false, process_d3d.Id);
            byte[] present_pattern = new byte[8];
            byte[] reset_pattern = new byte[8];
            foreach (System.Diagnostics.ProcessModule tesla in process_d3d.Modules)
            {
                
                if (tesla.FileName.Contains("d3d9.dll"))
                {
                    real_address = tesla.BaseAddress.ToInt64();
                }
            }
         
            uint kek;

            VirtualProtectEx(d3dProcessHandle, real_address + IDirectx9Device, (UIntPtr)((ulong)((long)8)), 0x80, out kek);
            
            ReadProcessMemory(d3dProcessHandle, real_address + IDirectx9Device, present_pattern, 8,0);
            VirtualProtectEx(d3dProcessHandle, real_address + IDirectx9Device, (UIntPtr)((ulong)((long)1)), 0x10, out kek);

            WriteProcessMemory(d3dProcessHandle, real_address + IDirectx9Device, new byte[] { 0xE9,  0x00, 0x00, 0x08, 0xAA }, 5, 0);
            cout("old pattern present: " + ByteArrayToString(present_pattern), 0);
            ReadProcessMemory(d3dProcessHandle, real_address + IDirectx9Device, present_pattern, 8, 0);
            cout("new pattern present: " + ByteArrayToString(present_pattern), 0);
            ReadProcessMemory(d3dProcessHandle, real_address + IDirectx9DeviceReset, reset_pattern, 8, 0);
            cout("old pattern reset: " + ByteArrayToString(reset_pattern), 0);

            WriteProcessMemory(d3dProcessHandle, real_address + IDirectx9DeviceReset, new byte[] { 0xE9, 0x00, 0x00, 0x08, 0xAA }, 5, 0);
            ReadProcessMemory(d3dProcessHandle, real_address + IDirectx9DeviceReset, reset_pattern, 8, 0);
            cout("new pattern reset: " + ByteArrayToString(reset_pattern), 0);
            Console.ForegroundColor = ConsoleColor.Green;
            cout("D3D9 Hook successfully written. but it may crash your process because this hook is still under developement.",1);
         
        }
        static void Main(string[] args)
        {
            x64 = true; //you can change this if you are using a system which kernel based on win32.

            Implement_d3d_device(0x5880,"Growtopia");
            cout("test", 0);
        }
    }
}

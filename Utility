using System;
using System.Collections.Generic;
using System.Text;
using LuaInterface;
using LitJson;

public class Utility 
{
    ////////////////////////////////////////////////////////////////////////////////////////
    /// <summary>
    /// 根据config配置列表将LuaTable转成byte数组
    /// </summary>
    public static byte[] ToBytesByTable(LuaTable source, string config)
    {
        JsonData jsonData = JsonMapper.ToObject(config);
        List<KeyValuePair<string, object>> result = ParseConfigJson(jsonData);
        List<byte> bytes = ParseKeyValuePairList(result, source);
        return bytes.ToArray();
    }
    /// <summary>
    /// 根据config配置列表将LuaTable转成byte数组
    /// </summary>
    public static LuaTable ToTableByBytes(byte[] bytes, string config, LuaTable table)
    {
        JsonData jsonData = JsonMapper.ToObject(config);
        List<KeyValuePair<string, object>> result = ParseConfigJson(jsonData);
        int seek = 0;
        ParseKeyValuePairList(table, result, bytes, ref seek);
        return table;
    }
    #region 解析代码
    private static List<byte> ParseKeyValuePairList(List<KeyValuePair<string, object>> result, LuaTable table)
    {
        List<byte> bytes = new List<byte>();
        for (int i = 0; i < result.Count; i++)
        {
            KeyValuePair<string, object> kv = result[i];
            if (kv.Value is TargetInfo)
            {
                TargetInfo info = (TargetInfo)kv.Value;
                Type type = info.type;
                if (info.isArray)
                {
                    byte[] tmp;
                    if (type == typeof(string))
                    {
                        string str = Convert.ToString(table[kv.Key]);
                        byte[] data = Encoding.Unicode.GetBytes(str);
                        tmp = new byte[info.Length * sizeof(char)];
                        Array.Copy(data, tmp, data.Length);
                        bytes.AddRange(tmp);
                    }
                    else
                    {
                        LuaTable child = (LuaTable)table[kv.Key];
                        for (int j = 1; j <= info.Length; j++)
                        {
                            if (j > result.Count)
                            {
                                bytes.AddRange(ParseToBytes(type, 0));
                            }
                            else
                            {
                                bytes.AddRange(ParseToBytes(type, child[j]));
                            }
                        }
                    }
                }
                else
                {
                    bytes.AddRange(ParseToBytes(type, table[kv.Key]));
                }
            }
            else
            {
                bytes.AddRange(ParseKeyValuePairList((List<KeyValuePair<string, object>>)kv.Value, (LuaTable)table[kv.Key]));
            }
        }
        return bytes;
    }
    private static LuaTable ParseKeyValuePairList(LuaTable table, List<KeyValuePair<string, object>> result, byte[] bytes, ref int seek)
    {
        for (int i = 0; i < result.Count; i++)
        {
            KeyValuePair<string, object> kv = result[i];
            if (kv.Value is TargetInfo)
            {
                TargetInfo info = (TargetInfo)kv.Value;
                Type type = info.type;
                if (info.isArray)
                {
                    if (type == typeof(string))
                    {
                        table[kv.Key] = Encoding.Unicode.GetString(bytes, seek, info.Length);
                        seek += info.Length * sizeof(char);
                    }
                    else
                    {
                        LuaTable child = (LuaTable)table[kv.Key];
                        for (int j = 1; j <= info.Length; j++)
                        {
                            child[j] = ParseToObject(type, bytes, ref seek);
                        }
                    }
                }
                else
                {
                    table[kv.Key] = ParseToObject(type, bytes, ref seek);
                }

            }
            else
            {
                ParseKeyValuePairList((LuaTable)table[kv.Key], (List<KeyValuePair<string, object>>)kv.Value, bytes, ref seek);
            }
        }
        return table;
    }
    private static byte[] ParseToBytes(Type type, object obj)
    {
        if (type == typeof(byte))
        {
            return new byte[] { Convert.ToByte(obj) };
        }
        else if (type == typeof(char))
        {
            return BitConverter.GetBytes(Convert.ToChar(obj));
        }
        else if (type == typeof(bool))
        {
            return BitConverter.GetBytes(Convert.ToBoolean(obj));
        }
        else if (type == typeof(short))
        {
            return BitConverter.GetBytes(Convert.ToInt16(obj));
        }
        else if (type == typeof(ushort))
        {
            return BitConverter.GetBytes(Convert.ToUInt16(obj));
        }
        else if (type == typeof(int))
        {
            return BitConverter.GetBytes(Convert.ToInt32(obj));
        }
        else if (type == typeof(uint))
        {
            return BitConverter.GetBytes(Convert.ToUInt32(obj));
        }
        else if (type == typeof(long))
        {
            return BitConverter.GetBytes(Convert.ToInt64(obj));
        }
        else if (type == typeof(ulong))
        {
            return BitConverter.GetBytes(Convert.ToUInt64(obj));
        }
        else if (type == typeof(float))
        {
            return BitConverter.GetBytes(Convert.ToSingle(obj));
        }
        else if (type == typeof(double))
        {
            return BitConverter.GetBytes((double)obj);
        }
        return null;
    }
    private static object ParseToObject(Type type, byte[] bytes, ref int seek)
    {
        object result = null;
        if (type == typeof(byte))
        {
            result = bytes[seek++];
        }
        else if (type == typeof(char))
        {
            result = BitConverter.ToChar(bytes, seek).ToString();
            seek += sizeof(char);
        }
        else if (type == typeof(bool))
        {
            result = BitConverter.ToBoolean(bytes, seek);
            seek += sizeof(bool);
        }
        else if (type == typeof(short))
        {
            result = BitConverter.ToInt16(bytes, seek);
            seek += sizeof(short);
        }
        else if (type == typeof(ushort))
        {
            result = BitConverter.ToUInt16(bytes, seek);
            seek += sizeof(short);
        }
        else if (type == typeof(int))
        {
            result = BitConverter.ToInt32(bytes, seek);
            seek += sizeof(int);
        }
        else if (type == typeof(uint))
        {
            result = BitConverter.ToUInt32(bytes, seek);
            seek += sizeof(int);
        }
        else if (type == typeof(long))
        {
            result = BitConverter.ToInt64(bytes, seek);
            seek += sizeof(long);
        }
        else if (type == typeof(ulong))
        {
            result = BitConverter.ToUInt64(bytes, seek);
            seek += sizeof(long);
        }
        else if (type == typeof(float))
        {
            result = BitConverter.ToSingle(bytes, seek);
            seek += sizeof(float);
        }
        else if (type == typeof(double))
        {
            result = BitConverter.ToDouble(bytes, seek);
            seek += sizeof(double);
        }
        return result;
    }
    private static List<KeyValuePair<string, object>> ParseConfigJson(JsonData jsonData)
    {
        List<KeyValuePair<string, object>> list = new List<KeyValuePair<string, object>>();
        IEnumerator<string> keys = jsonData.Keys.GetEnumerator();
        while (keys.MoveNext())
        {
            string key = keys.Current;
            JsonData value = jsonData[keys.Current];
            if (value.IsObject)
            {
                list.Add(new KeyValuePair<string, object>(key, ParseConfigJson((JsonData)value)));
            }
            else
            {
                TargetInfo result = GetTargetInfoByString((string)value);
                if (result.isArray != null)
                {
                    list.Add(new KeyValuePair<string, object>(key, result));
                }
            }
        }
        return list;
    }
    private struct TargetInfo
    {
        public Type type;
        public int Length;
        public bool isArray;
    }
    private static TargetInfo GetTargetInfoByString(string type)
    {
        TargetInfo target = new TargetInfo();
        if (type[type.Length - 1] == ']')
        {
            int index = type.IndexOf('[');
            int.TryParse(type.Substring(index + 1, type.Length - index - 2), out target.Length);
            type = type.Substring(0, index);
            target.isArray = true;
        }
        type = type.ToLower();
        switch (type)
        {
            case "byte": target.type = typeof(byte); break;
            case "char": target.type = target.isArray ? typeof(string) : typeof(char); break;
            case "bool":
            case "boolean": target.type = typeof(bool); break;
            case "short":
            case "int16": target.type = typeof(short); break;
            case "ushort":
            case "uint16": target.type = typeof(ushort); break;
            case "int":
            case "int32": target.type = typeof(int); break;
            case "uint":
            case "uint32": target.type = typeof(uint); break;
            case "long":
            case "int64": target.type = typeof(long); break;
            case "ulong":
            case "uint64": target.type = typeof(ulong); break;
            case "float":
            case "single": target.type = typeof(float); break;
            case "double": target.type = typeof(double); break;
        }
        return target;
    }
    #endregion
}

public static final int UPPERCASE = 1;  // 0001
public static final int REVERSE   = 2;  // 0010
public static final int FULL_STOP = 4;  // 0100
public static final int EMPHASISE = 8;  // 1000
public static final int ALL_OPTS  = 15; // 1111

public static String format(String value, int flags)
{
    if ((flags & UPPERCASE) == UPPERCASE) value = value.toUpperCase();

    if ((flags & REVERSE) == REVERSE) value = new StringBuffer(value).reverse().toString();

    if ((flags & FULL_STOP) == FULL_STOP) value += ".";

    if ((flags & EMPHASISE) == EMPHASISE) value = "~*~ " + value + " ~*~";

    return value;
}



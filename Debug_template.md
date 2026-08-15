class D {
    static void p(Object... o) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < o.length; i += 2) {
            String val = (i + 1 < o.length) ? Arrays.deepToString(new Object[]{o[i + 1]}) : "[]";
            sb.append(o[i]).append(": ").append(val.substring(1, val.length() - 1)).append("  |  ");
        }
        System.out.println(sb);
    }
}

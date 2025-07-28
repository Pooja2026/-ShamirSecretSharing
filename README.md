import java.math.BigInteger;
import java.util.*;
import java.util.regex.Pattern;
import java.util.regex.Matcher;

class Shamir {

    static class Point {
        BigInteger x, y;

        Point(BigInteger x, BigInteger y) {
            this.x = x;
            this.y = y;
        }
    }

    static class ShareData {
        String base;
        String value;

        ShareData(String base, String value) {
            this.base = base;
            this.value = value;
        }
    }

    // Simple JSON parser for our specific structure
    static class SimpleJSONParser {
        private String json;

        SimpleJSONParser(String json) {
            this.json = json.replaceAll("\\s+", ""); // Remove whitespace
        }

        int getK() {
            Pattern pattern = Pattern.compile("\"k\":(\\d+)");
            Matcher matcher = pattern.matcher(json);
            if (matcher.find()) {
                return Integer.parseInt(matcher.group(1));
            }
            throw new RuntimeException("Could not find 'k' value in JSON");
        }

        Map<String, ShareData> getShares() {
            Map<String, ShareData> shares = new HashMap<>();

            // Pattern to match share objects like "1":{"base":"10","value":"4"}
            Pattern sharePattern = Pattern.compile("\"(\\d+)\":\\{\"base\":\"(\\d+)\",\"value\":\"([^\"]+)\"\\}");
            Matcher matcher = sharePattern.matcher(json);

            while (matcher.find()) {
                String shareId = matcher.group(1);
                String base = matcher.group(2);
                String value = matcher.group(3);
                shares.put(shareId, new ShareData(base, value));
            }

            return shares;
        }
    }

    // Decode a string in given base to BigInteger
    static BigInteger decodeInBase(String val, int base) {
        return new BigInteger(val, base);
    }

    // Compute Lagrange interpolation at x=0: f(0) using regular arithmetic
    static BigInteger lagrangeAtZero(List<Point> points) {
        BigInteger result = BigInteger.ZERO;
        int k = points.size();

        for (int j = 0; j < k; j++) {
            BigInteger xj = points.get(j).x;
            BigInteger yj = points.get(j).y;
            BigInteger num = BigInteger.ONE;
            BigInteger den = BigInteger.ONE;

            for (int m = 0; m < k; m++) {
                if (m == j)
                    continue;
                BigInteger xm = points.get(m).x;
                // Calculate (0 - xm) = -xm
                num = num.multiply(xm.negate());
                // Calculate (xj - xm)
                den = den.multiply(xj.subtract(xm));
            }

            // Calculate yj * num / den
            BigInteger term = yj.multiply(num).divide(den);
            result = result.add(term);
        }

        return result;
    }

    static BigInteger solveTestCase(String jsonInput) {
        // Parse JSON
        SimpleJSONParser parser = new SimpleJSONParser(jsonInput);
        int k = parser.getK();
        Map<String, ShareData> sharesData = parser.getShares();

        // Convert to points
        List<Point> pts = new ArrayList<>();
        for (Map.Entry<String, ShareData> entry : sharesData.entrySet()) {
            String key = entry.getKey();
            ShareData shareData = entry.getValue();

            int base = Integer.parseInt(shareData.base);
            String valstr = shareData.value;
            BigInteger y = decodeInBase(valstr, base);
            BigInteger x = new BigInteger(key);
            pts.add(new Point(x, y));
        }

        if (pts.size() < k) {
            return BigInteger.ZERO;
        }

        // Use the first k points to solve for the polynomial
        List<Point> selectedPoints = pts.subList(0, k);
        return lagrangeAtZero(selectedPoints);
    }

    public static void main(String[] args) throws Exception {
        // Test Case 1
        String testCase1 = "{\n" +
                "    \"keys\": {\n" +
                "        \"n\": 4,\n" +
                "        \"k\": 3\n" +
                "    },\n" +
                "    \"1\": {\n" +
                "        \"base\": \"10\",\n" +
                "        \"value\": \"4\"\n" +
                "    },\n" +
                "    \"2\": {\n" +
                "        \"base\": \"2\",\n" +
                "        \"value\": \"111\"\n" +
                "    },\n" +
                "    \"3\": {\n" +
                "        \"base\": \"10\",\n" +
                "        \"value\": \"12\"\n" +
                "    },\n" +
                "    \"6\": {\n" +
                "        \"base\": \"4\",\n" +
                "        \"value\": \"213\"\n" +
                "    }\n" +
                "}";

        // Test Case 2
        String testCase2 = "{\n" +
                "    \"keys\": {\n" +
                "        \"n\": 10,\n" +
                "        \"k\": 7\n" +
                "    },\n" +
                "    \"1\": {\n" +
                "        \"base\": \"6\",\n" +
                "        \"value\": \"13444211440455345511\"\n" +
                "    },\n" +
                "    \"2\": {\n" +
                "        \"base\": \"15\",\n" +
                "        \"value\": \"aed7015a346d63\"\n" +
                "    },\n" +
                "    \"3\": {\n" +
                "        \"base\": \"15\",\n" +
                "        \"value\": \"6aeeb69631c227c\"\n" +
                "    },\n" +
                "    \"4\": {\n" +
                "        \"base\": \"16\",\n" +
                "        \"value\": \"e1b5e05623d881f\"\n" +
                "    },\n" +
                "    \"5\": {\n" +
                "        \"base\": \"8\",\n" +
                "        \"value\": \"316034514573652620673\"\n" +
                "    },\n" +
                "    \"6\": {\n" +
                "        \"base\": \"3\",\n" +
                "        \"value\": \"2122212201122002221120200210011020220200\"\n" +
                "    },\n" +
                "    \"7\": {\n" +
                "        \"base\": \"3\",\n" +
                "        \"value\": \"20120221122211000100210021102001201112121\"\n" +
                "    },\n" +
                "    \"8\": {\n" +
                "        \"base\": \"6\",\n" +
                "        \"value\": \"20220554335330240002224253\"\n" +
                "    },\n" +
                "    \"9\": {\n" +
                "        \"base\": \"12\",\n" +
                "        \"value\": \"45153788322a1255483\"\n" +
                "    },\n" +
                "    \"10\": {\n" +
                "        \"base\": \"7\",\n" +
                "        \"value\": \"1101613130313526312514143\"\n" +
                "    }\n" +
                "}";

        // Solve both test cases
        BigInteger secret1 = solveTestCase(testCase1);
        BigInteger secret2 = solveTestCase(testCase2);

        System.out.println("Secret for Test Case 1: " + secret1);
        System.out.println("Secret for Test Case 2: " + secret2);
    }
}

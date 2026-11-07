import javax.swing.*;
import java.awt.*;
import java.util.*;
import java.util.List;

public class Main extends JFrame {

    // 1. Sensor class
    static class Sensor {
        int id;
        double x, y;

        Sensor(int id, double x, double y) {
            this.id = id;
            this.x = x;
            this.y = y;
        }

        public String toString() {
            return "S" + id + " (" + x + ", " + y + ")";
        }
    }

    // 2. Edge class
    static class Edge implements Comparable<Edge> {
        Sensor source, destination;
        double weight;

        Edge(Sensor a, Sensor b) {
            source = a;
            destination = b;
            weight = distance(a, b);
        }

        public int compareTo(Edge e) {
            return Double.compare(weight, e.weight);
        }
    }

    // 3. Calculate Euclidean distance
    static double distance(Sensor a, Sensor b) {
        double dx = a.x - b.x;
        double dy = a.y - b.y;
        return Math.sqrt(dx * dx + dy * dy);
    }

    // 4. Cross product
    static double cross(Sensor a, Sensor b, Sensor c) {
        return (b.x - a.x) * (c.y - a.y)
             - (b.y - a.y) * (c.x - a.x);
    }

    // 5. Graham Scan - Convex Hull
    static List<Sensor> convexHull(List<Sensor> sensors) {

        Map<String, Sensor> unique = new LinkedHashMap<>();

        for (Sensor s : sensors) {
            unique.putIfAbsent(
                Double.toString(s.x) + "," + Double.toString(s.y), s);
        }

        List<Sensor> points = new ArrayList<>(unique.values());

        if (points.size() <= 1)
            return points;

        Sensor pivot = points.stream()
            .min(Comparator.comparingDouble((Sensor s) -> s.y)
            .thenComparingDouble(s -> s.x))
            .get();

        points.remove(pivot);

        points.sort((a, b) -> {
            double c = cross(pivot, a, b);

            if (Math.abs(c) < 1e-10)
                return Double.compare(distance(pivot, a),
                                      distance(pivot, b));

            return c > 0 ? -1 : 1;
        });

        Stack<Sensor> stack = new Stack<>();
        stack.push(pivot);

        for (Sensor p : points) {

            while (stack.size() >= 2 &&
                   cross(stack.get(stack.size() - 2),
                         stack.peek(), p) <= 1e-10) {
                stack.pop();
            }

            stack.push(p);
        }

        return new ArrayList<>(stack);
    }

    // 6. Union Find
    static class UnionFind {
        int[] parent, rank;

        UnionFind(int n) {
            parent = new int[n];
            rank = new int[n];

            for (int i = 0; i < n; i++)
                parent[i] = i;
        }

        int find(int x) {
            if (parent[x] != x)
                parent[x] = find(parent[x]);

            return parent[x];
        }

        boolean union(int a, int b) {
            int rootA = find(a);
            int rootB = find(b);

            if (rootA == rootB)
                return false;

            if (rank[rootA] < rank[rootB])
                parent[rootA] = rootB;

            else if (rank[rootA] > rank[rootB])
                parent[rootB] = rootA;

            else {
                parent[rootB] = rootA;
                rank[rootA]++;
            }

            return true;
        }
    }

    // 7. Kruskal MST
    static List<Edge> kruskalMST(List<Sensor> sensors) {

        List<Edge> edges = new ArrayList<>();
        List<Edge> mst = new ArrayList<>();

        int n = sensors.size();

        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                edges.add(new Edge(sensors.get(i),
                                   sensors.get(j)));
            }
        }

        Collections.sort(edges);

        UnionFind uf = new UnionFind(n);

        Map<Integer, Integer> index = new HashMap<>();

        for (int i = 0; i < n; i++)
            index.put(sensors.get(i).id, i);

        for (Edge e : edges) {

            int u = index.get(e.source.id);
            int v = index.get(e.destination.id);

            if (uf.union(u, v)) {
                mst.add(e);

                if (mst.size() == n - 1)
                    break;
            }
        }

        return mst;
    }

    // 8. Calculate MST cost
    static double totalDistance(List<Edge> mst) {
        double total = 0;

        for (Edge e : mst)
            total += e.weight;

        return total;
    }

    // 9. Visualization panel
    static class NetworkPanel extends JPanel {

        List<Sensor> sensors;
        List<Sensor> hull;
        List<Edge> mst;

        NetworkPanel(List<Sensor> s,
                     List<Sensor> h,
                     List<Edge> m) {
            sensors = s;
            hull = h;
            mst = m;
            setBackground(Color.WHITE);
        }

        protected void paintComponent(Graphics g) {
            super.paintComponent(g);

            if (sensors.isEmpty())
                return;

            Graphics2D g2 = (Graphics2D) g;

            g2.setRenderingHint(
                RenderingHints.KEY_ANTIALIASING,
                RenderingHints.VALUE_ANTIALIAS_ON);

            double minX = sensors.stream()
                .mapToDouble(s -> s.x).min().getAsDouble();

            double maxX = sensors.stream()
                .mapToDouble(s -> s.x).max().getAsDouble();

            double minY = sensors.stream()
                .mapToDouble(s -> s.y).min().getAsDouble();

            double maxY = sensors.stream()
                .mapToDouble(s -> s.y).max().getAsDouble();

            double rangeX = Math.max(maxX - minX, 1);
            double rangeY = Math.max(maxY - minY, 1);

            int margin = 55;

            double scale = Math.min(
                (getWidth() - 2.0 * margin) / rangeX,
                (getHeight() - 2.0 * margin) / rangeY);

            double offsetX =
                (getWidth() - rangeX * scale) / 2;

            double offsetY =
                (getHeight() - rangeY * scale) / 2;

            java.util.function.Function<Sensor, Point> pos = s ->
                new Point(
                    (int)(offsetX + (s.x - minX) * scale),
                    (int)(getHeight() - offsetY
                          - (s.y - minY) * scale));

            // Draw convex hull
            g2.setColor(Color.BLUE);
            g2.setStroke(new BasicStroke(2));

            if (hull.size() >= 2) {
                for (int i = 0; i < hull.size(); i++) {
                    Point a = pos.apply(hull.get(i));
                    Point b = pos.apply(
                        hull.get((i + 1) % hull.size()));

                    g2.drawLine(a.x, a.y, b.x, b.y);
                }
            }

            // Draw MST
            g2.setColor(new Color(0, 150, 70));
            g2.setStroke(new BasicStroke(3));

            for (Edge e : mst) {
                Point a = pos.apply(e.source);
                Point b = pos.apply(e.destination);

                g2.drawLine(a.x, a.y, b.x, b.y);
            }

            // Draw sensors
            for (Sensor s : sensors) {
                Point p = pos.apply(s);

                g2.setColor(Color.RED);
                g2.fillOval(p.x - 7, p.y - 7, 14, 14);

                g2.setColor(Color.BLACK);
                g2.drawString("S" + s.id, p.x + 9, p.y - 9);
            }

            g2.setColor(Color.BLACK);
            g2.drawString(
                "Blue: Convex Hull | Green: MST | Red: Sensors",
                15, 20);
        }
    }

    // 10. GUI components
    JTextArea inputArea, outputArea;
    JPanel visualPanel;

    Main() {
        setTitle("AirGuard - Air Quality Monitoring Network");
        setSize(1100, 750);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);

        JLabel title = new JLabel(
            "AIRGUARD - MONITORING NETWORK",
            JLabel.CENTER);

        title.setFont(new Font("Arial", Font.BOLD, 22));

        inputArea = new JTextArea(
            "1,1\n2,4\n4,5\n6,3\n5,1\n3,2");

        inputArea.setFont(new Font("Monospaced", Font.PLAIN, 15));

        JButton calculateButton =
            new JButton("Generate Network");

        JButton sampleButton =
            new JButton("Load Sample Data");

        outputArea = new JTextArea(9, 30);
        outputArea.setEditable(false);
        outputArea.setFont(new Font("Monospaced", Font.PLAIN, 13));

        visualPanel = new JPanel(new BorderLayout());
        visualPanel.setBackground(Color.WHITE);

        calculateButton.addActionListener(e -> calculate());

        sampleButton.addActionListener(e -> {
            inputArea.setText(
                "1,1\n2,4\n4,5\n6,3\n5,1\n3,2");
            calculate();
        });

        JPanel buttons = new JPanel();
        buttons.add(calculateButton);
        buttons.add(sampleButton);

        JPanel leftPanel = new JPanel(new BorderLayout());

        leftPanel.add(
            new JLabel("Enter Sensor Coordinates (X,Y)"),
            BorderLayout.NORTH);

        leftPanel.add(new JScrollPane(inputArea),
                      BorderLayout.CENTER);

        leftPanel.add(buttons, BorderLayout.SOUTH);

        JPanel rightPanel = new JPanel(new BorderLayout());

        rightPanel.add(
            new JLabel("Network Visualization"),
            BorderLayout.NORTH);

        rightPanel.add(visualPanel, BorderLayout.CENTER);

        JSplitPane split = new JSplitPane(
            JSplitPane.HORIZONTAL_SPLIT,
            leftPanel, rightPanel);

        split.setDividerLocation(260);

        add(title, BorderLayout.NORTH);
        add(split, BorderLayout.CENTER);
        add(new JScrollPane(outputArea), BorderLayout.SOUTH);

        calculate();
    }

    // 11. Main calculation
    void calculate() {
        try {
            List<Sensor> sensors = new ArrayList<>();

            String[] lines =
                inputArea.getText().trim().split("\\R");

            int id = 1;

            for (String line : lines) {

                if (line.trim().isEmpty())
                    continue;

                String[] values = line.split(",");

                if (values.length != 2)
                    throw new IllegalArgumentException(
                        "Invalid coordinate: " + line);

                double x = Double.parseDouble(values[0].trim());
                double y = Double.parseDouble(values[1].trim());

                if (!Double.isFinite(x) || !Double.isFinite(y))
                    throw new IllegalArgumentException(
                        "Coordinates must be finite.");

                sensors.add(new Sensor(id++, x, y));
            }

            if (sensors.isEmpty())
                throw new IllegalArgumentException(
                    "Enter at least one sensor.");

            List<Sensor> hull = convexHull(sensors);
            List<Edge> mst = kruskalMST(sensors);

            double total = totalDistance(mst);

            StringBuilder report = new StringBuilder();

            report.append("===== AIRGUARD REPORT =====\n\n");

            report.append("Total Sensors: ")
                  .append(sensors.size()).append("\n");

            report.append("Unique Locations: ")
                  .append(new HashSet<>(
                      sensors.stream()
                          .map(s -> s.x + "," + s.y)
                          .toList()).size())
                  .append("\n\n");

            report.append("CONVEX HULL BOUNDARY:\n");

            for (Sensor s : hull)
                report.append(s).append("\n");

            report.append("\nMST CONNECTIONS:\n");

            for (Edge e : mst) {
                report.append(e.source.id)
                      .append(" -- ")
                      .append(e.destination.id)
                      .append(" : ")
                      .append(String.format("%.2f", e.weight))
                      .append("\n");
            }

            report.append("\nTotal MST Distance: ")
                  .append(String.format("%.2f", total));

            double costPerUnit = 1000;

            report.append("\nEstimated Installation Cost: Rs. ")
                  .append(String.format("%.2f",
                      total * costPerUnit));

            report.append("\nCost Rate: Rs. 1000 per distance unit");

            report.append("\nNetwork Status: ")
                  .append("Connected");

            report.append("\n\nAlgorithms Used:");
            report.append("\n1. Graham Scan");
            report.append("\n2. Kruskal MST");
            report.append("\n3. Union-Find");
            report.append("\n4. Euclidean Distance");

            report.append("\n\nTime Complexities:");
            report.append("\nConvex Hull: O(n log n)");
            report.append("\nKruskal: O(E log E)");
            report.append("\nDistance: O(1) per pair");

            outputArea.setText(report.toString());

            visualPanel.removeAll();

            visualPanel.add(
                new NetworkPanel(sensors, hull, mst),
                BorderLayout.CENTER);

            visualPanel.revalidate();
            visualPanel.repaint();

        } catch (NumberFormatException ex) {
            JOptionPane.showMessageDialog(
                this, "Please enter valid numeric coordinates.");

        } catch (IllegalArgumentException ex) {
            JOptionPane.showMessageDialog(
                this, ex.getMessage());
        }
    }

    // 12. Program execution
    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            new Main().setVisible(true);
        });
    }
}

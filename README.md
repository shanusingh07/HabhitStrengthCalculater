import java.io.*;
import java.text.SimpleDateFormat;
import java.util.*;

class Habit {
    private String name;
    private String description;
    private String frequency;
    private List<Boolean> completions;
    private List<String> dates;

    public Habit(String name, String description, String frequency) {
        this.name = name;
        this.description = description;
        this.frequency = frequency;
        this.completions = new ArrayList<>();
        this.dates = new ArrayList<>();
    }

    public void addCompletion(boolean completed) {
        completions.add(completed);
        dates.add(new SimpleDateFormat("yyyy-MM-dd").format(new Date()));
    }

    public double calculateStrength() {
        if (completions.isEmpty()) return 0.0;
        
        int completedCount = 0;
        for (boolean completed : completions) {
            if (completed) completedCount++;
        }
        
        return ((double) completedCount / completions.size()) * 100;
    }

    public int getCurrentStreak() {
        if (completions.isEmpty() || !completions.get(completions.size() - 1)) {
            return 0;
        }
        
        int streak = 0;
        for (int i = completions.size() - 1; i >= 0; i--) {
            if (completions.get(i)) {
                streak++;
            } else {
                break;
            }
        }
        return streak;
    }

    public String getName() { return name; }
    public String getDescription() { return description; }
    public String getFrequency() { return frequency; }
    public List<Boolean> getCompletions() { return completions; }
    public List<String> getDates() { return dates; }

    public String toString() {
        return String.format("%s (%s): %.1f%% strength, Current streak: %d days", 
            name, description, calculateStrength(), getCurrentStreak());
    }
}

public class HabitStrengthCalculator {
    private static List<Habit> habits = new ArrayList<>();
    private static final String DATA_FILE = "habits_data.txt";

    public static void main(String[] args) {
        loadHabitsFromFile();
        
        Scanner scanner = new Scanner(System.in);
        boolean running = true;
        
        while (running) {
            System.out.println("\nHabit Strength Calculator");
            System.out.println("1. Create new habit");
            System.out.println("2. Track habit completion");
            System.out.println("3. View habit strengths");
            System.out.println("4. View habit history");
            System.out.println("5. Exit");
            System.out.print("Choose an option: ");
            
            int choice = scanner.nextInt();
            scanner.nextLine(); // consume newline
            
            switch (choice) {
                case 1:
                    createNewHabit(scanner);
                    break;
                case 2:
                    trackHabitCompletion(scanner);
                    break;
                case 3:
                    viewHabitStrengths();
                    break;
                case 4:
                    viewHabitHistory();
                    break;
                case 5:
                    running = false;
                    break;
                default:
                    System.out.println("Invalid option. Please try again.");
            }
        }
        
        saveHabitsToFile();
        System.out.println("Goodbye! Your habit data has been saved.");
    }

    private static void createNewHabit(Scanner scanner) {
        System.out.print("Enter habit name: ");
        String name = scanner.nextLine();
        
        System.out.print("Enter habit description: ");
        String description = scanner.nextLine();
        
        System.out.print("Enter frequency (Daily/Weekly): ");
        String frequency = scanner.nextLine();
        
        Habit habit = new Habit(name, description, frequency);
        habits.add(habit);
        System.out.println("Habit created successfully!");
    }

    private static void trackHabitCompletion(Scanner scanner) {
        if (habits.isEmpty()) {
            System.out.println("No habits available. Please create a habit first.");
            return;
        }
        
        System.out.println("Select a habit to track:");
        for (int i = 0; i < habits.size(); i++) {
            System.out.printf("%d. %s%n", i + 1, habits.get(i).getName());
        }
        
        System.out.print("Enter habit number: ");
        int habitIndex = scanner.nextInt() - 1;
        scanner.nextLine(); // consume newline
        
        if (habitIndex < 0 || habitIndex >= habits.size()) {
            System.out.println("Invalid habit number.");
            return;
        }
        
        System.out.print("Did you complete this habit today? (yes/no): ");
        String response = scanner.nextLine().toLowerCase();
        boolean completed = response.equals("yes");
        
        habits.get(habitIndex).addCompletion(completed);
        
        System.out.println("Completion recorded!");
        System.out.println(getFeedback(habits.get(habitIndex).calculateStrength()));
    }

    private static void viewHabitStrengths() {
        if (habits.isEmpty()) {
            System.out.println("No habits available.");
            return;
        }
        
        System.out.println("\nHabit Strengths:");
        for (Habit habit : habits) {
            double strength = habit.calculateStrength();
            System.out.printf("%s: %.1f%% - %s%n", 
                habit.getName(), 
                strength,
                getFeedback(strength));
        }
    }

    private static void viewHabitHistory() {
        if (habits.isEmpty()) {
            System.out.println("No habits available.");
            return;
        }
        
        System.out.println("\nHabit History:");
        for (Habit habit : habits) {
            System.out.println(habit);
            List<String> dates = habit.getDates();
            List<Boolean> completions = habit.getCompletions();
            
            for (int i = 0; i < dates.size(); i++) {
                System.out.printf("  %s: %s%n", 
                    dates.get(i), 
                    completions.get(i) ? "Completed" : "Missed");
            }
        }
    }

    private static String getFeedback(double strength) {
        if (strength >= 80) {
            return "Great job! You're consistently keeping up with your habit.";
        } else if (strength >= 50) {
            return "Good job! You're making progress, but there's room for improvement.";
        } else {
            return "Keep going! Try to be more consistent in completing your habit.";
        }
    }

    private static void saveHabitsToFile() {
        try (PrintWriter writer = new PrintWriter(new FileWriter(DATA_FILE))) {
            for (Habit habit : habits) {
                writer.println(habit.getName());
                writer.println(habit.getDescription());
                writer.println(habit.getFrequency());
                
                List<String> dates = habit.getDates();
                List<Boolean> completions = habit.getCompletions();
                
                writer.println(dates.size());
                for (int i = 0; i < dates.size(); i++) {
                    writer.println(dates.get(i));
                    writer.println(completions.get(i));
                }
            }
        } catch (IOException e) {
            System.out.println("Error saving habit data: " + e.getMessage());
        }
    }

    private static void loadHabitsFromFile() {
        File file = new File(DATA_FILE);
        if (!file.exists()) return;
        
        try (Scanner fileScanner = new Scanner(file)) {
            while (fileScanner.hasNextLine()) {
                String name = fileScanner.nextLine();
                String description = fileScanner.nextLine();
                String frequency = fileScanner.nextLine();
                
                Habit habit = new Habit(name, description, frequency);
                int entries = Integer.parseInt(fileScanner.nextLine());
                
                for (int i = 0; i < entries; i++) {
                    String date = fileScanner.nextLine();
                    boolean completed = Boolean.parseBoolean(fileScanner.nextLine());
                    habit.getDates().add(date);
                    habit.getCompletions().add(completed);
                }
                
                habits.add(habit);
            }
        } catch (IOException e) {
            System.out.println("Error loading habit data: " + e.getMessage());
        }
    }
}

# OOP-PROJECT-2-FINAL

# Criminal Agency System

// Fully GUI-based Prisoner Management System using SFML with Background and Music
#include <SFML/Graphics.hpp>
#include <SFML/Audio.hpp>
#include <SFML/Window.hpp>
#include <fstream>
#include <iostream>
#include <sstream>
#include <string>

using namespace std;

const int MAX_PROFILES = 100;
const int MAX_CRIMES = 10;
const string DATA_FILE = "profiles.txt";

class Profile {
public:
    int id;
    string name, address, dob;
    string crimes[MAX_CRIMES];
    int crimeCount;

   Profile() : id(0), crimeCount(0) {}

  void setProfile(int pid, const string& pname, const string& paddress, const string& pdob) {
        id = pid; name = pname; address = paddress; dob = pdob; crimeCount = 0;
    }

  void addCrime(const string& crime) {
        if (crimeCount < MAX_CRIMES) crimes[crimeCount++] = crime;
    }
    string getDisplayText() const {
        string output = "ID: " + to_string(id) + "\nName: " + name + "\nAddress: " + address + "\nDOB: " + dob + "\nCrimes:\n";
        for (int i = 0; i < crimeCount; ++i) output += "  - " + crimes[i] + "\n";
        output += (crimeCount >= 10) ? "Status: Bail not possible. Punishment: Life imprisonment.\n" :
            (crimeCount > 0) ? "Status: Bail possible. Punishment: " + to_string(crimeCount) + " year(s).\n" :
            "Status: No crimes recorded.\n";
        return output;
    }
};

class AgencySystem {
public:
    Profile profiles[MAX_PROFILES];
    int profileCount = 0, nextID = 1;
    void loadProfiles() {
        ifstream fin(DATA_FILE);
        if (!fin) return;
        profileCount = 0; nextID = 1;
        while (fin && profileCount < MAX_PROFILES) {
            Profile& p = profiles[profileCount]; string line;
            if (!getline(fin, line) || line.empty()) break;
            p.id = stoi(line);
            getline(fin, p.name); getline(fin, p.address); getline(fin, p.dob);
            getline(fin, line); p.crimeCount = stoi(line);
            for (int i = 0; i < p.crimeCount; ++i) getline(fin, p.crimes[i]);
            if (p.id >= nextID) nextID = p.id + 1; profileCount++;
        }
        fin.close();
    }
    void saveProfiles() {
        ofstream fout(DATA_FILE);
        for (int i = 0; i < profileCount; ++i) {
            Profile& p = profiles[i];
            fout << p.id << "\n" << p.name << "\n" << p.address << "\n" << p.dob << "\n" << p.crimeCount << "\n";
            for (int j = 0; j < p.crimeCount; ++j) fout << p.crimes[j] << "\n";
        }
        fout.close();
    }
    Profile* searchByID(int id) {
        for (int i = 0; i < profileCount; ++i) if (profiles[i].id == id) return &profiles[i];
        return nullptr;
    }
    bool deleteByID(int id) {
        for (int i = 0; i < profileCount; ++i) {
            if (profiles[i].id == id) {
                for (int j = i; j < profileCount - 1; ++j)
                    profiles[j] = profiles[j + 1];
                profileCount--;
                return true;
            }
        }
        return false;
    }
    void addProfile(const string& name, const string& address, const string& dob, const string crimes[], int crimeCount) {
        if (profileCount >= MAX_PROFILES) return;
        Profile& p = profiles[profileCount++];
        p.setProfile(nextID++, name, address, dob);
        for (int i = 0; i < crimeCount; ++i) p.addCrime(crimes[i]);
    }
};

int main() {
    AgencySystem system;
    system.loadProfiles();
    sf::RenderWindow window(sf::VideoMode(1000, 700), "Prisoner Management System");
    window.setFramerateLimit(60);
    sf::Font font;
    if (!font.loadFromFile("C:\\Users\\DELL\\OneDrive\\Desktop\\fu\\f.ttf")) {
        cout << "Failed to load font\n";
        return -1;
    }
    sf::Texture backgroundTexture;
    if (!backgroundTexture.loadFromFile("C:\\Users\\DELL\\OneDrive\\Desktop\\image.jpg")) {
        cout << "Failed to load background image\n";
        return -1;
    }
    sf::Sprite backgroundSprite(backgroundTexture);
    sf::Music music;
    if (!music.openFromFile("C:\\Users\\DELL\\OneDrive\\Desktop\\music.mp3")) {
        cout << "Failed to load music\n";
        return -1;
    }
    music.setLoop(true);
    music.play();
    sf::Text displayText("", font, 20);
    displayText.setFillColor(sf::Color::White);
    displayText.setPosition(250, 50);
    sf::String inputString;
    sf::Text inputText("", font, 20);
    inputText.setFillColor(sf::Color::Green);
    inputText.setPosition(250, 600);
    enum Mode { NONE, ADD, SEARCH, DELETE } mode = NONE;
    string name, address, dob, crime;
    int step = 0;
    string crimes[MAX_CRIMES];
    int crimeCount = 0;
    sf::RectangleShape buttons[4];
    sf::Text buttonLabels[4];
    string labels[4] = { "Add", "Search", "Delete", "Show All" };
    for (int i = 0; i < 4; ++i) {
        buttons[i].setSize(sf::Vector2f(120, 40));
        buttons[i].setPosition(50, 50 + i * 60);
        buttons[i].setFillColor(sf::Color(100, 100, 250));
        buttonLabels[i].setFont(font);
        buttonLabels[i].setCharacterSize(18);
        buttonLabels[i].setString(labels[i]);
        buttonLabels[i].setPosition(60, 58 + i * 60);
        buttonLabels[i].setFillColor(sf::Color::White);
    }
    while (window.isOpen()) {
        sf::Event e;
        while (window.pollEvent(e)) {
            if (e.type == sf::Event::Closed) window.close();
            if (e.type == sf::Event::TextEntered) {
                if (e.text.unicode == 8 && !inputString.isEmpty()) inputString.erase(inputString.getSize() - 1, 1);
                else if (e.text.unicode == 13 || e.text.unicode == 10) {
                    string input = inputString;
                    inputString = "";
                    try {
                        switch (mode) {
                        case ADD:
                            if (step == 0) { name = input; step++; displayText.setString("Enter Address:"); }
                            else if (step == 1) { address = input; step++; displayText.setString("Enter DOB:"); }
                            else if (step == 2) { dob = input; step++; displayText.setString("Enter Crime or type 'done':"); }
                            else if (step >= 3) {
                                if (input == "done" || crimeCount == MAX_CRIMES) {
                                    system.addProfile(name, address, dob, crimes, crimeCount);
                                    system.saveProfiles();
                                    displayText.setString("Profile Added!");
                                    step = crimeCount = 0; mode = NONE;
                                }
                                else {
                                    crimes[crimeCount++] = input;
                                    displayText.setString("Enter another Crime or type 'done':");
                                }
                            }
                            break;
                        case SEARCH:
                            if (Profile* p = system.searchByID(stoi(input)))
                                displayText.setString(p->getDisplayText());
                            else
                                displayText.setString("Not Found");
                            mode = NONE;
                            break;
                        case DELETE:
                            if (system.deleteByID(stoi(input))) {
                                system.saveProfiles();
                                displayText.setString("Profile Deleted");
                            }
                            else displayText.setString("Profile Not Found");
                            mode = NONE;
                            break;
                        default:
                            break;
                        }
                    }
                    catch (...) {
                        displayText.setString("Invalid input. Try again.");
                    }
                }
                else if (e.text.unicode < 128) {
                    inputString += static_cast<char>(e.text.unicode);
                }
                inputText.setString("Input: " + inputString);
            }
            if (e.type == sf::Event::MouseButtonPressed) {
                sf::Vector2f mouse(static_cast<float>(sf::Mouse::getPosition(window).x), static_cast<float>(sf::Mouse::getPosition(window).y));
                for (int i = 0; i < 4; ++i) {
                    if (buttons[i].getGlobalBounds().contains(mouse)) {
                        switch (i) {
                        case 0: mode = ADD; step = 0; crimeCount = 0; displayText.setString("Enter Name:"); break;
                        case 1: mode = SEARCH; displayText.setString("Enter ID to Search:"); break;
                        case 2: mode = DELETE; displayText.setString("Enter ID to Delete:"); break;
                        case 3: {
                            if (system.profileCount == 0) displayText.setString("No profiles available.");
                            else {
                                string allText;
                                for (int j = 0; j < system.profileCount; ++j)
                                    allText += system.profiles[j].getDisplayText() + "----------------------\n";
                                displayText.setString(allText);
                            }
                            break;
                        }
                        }
                    }
                }
            }
        }
        window.clear();
        window.draw(backgroundSprite);
        for (int i = 0; i < 4; ++i) {
            window.draw(buttons[i]);
            window.draw(buttonLabels[i]);
        }
        window.draw(displayText);
        window.draw(inputText);
        window.display();
    }
    return 0;
}


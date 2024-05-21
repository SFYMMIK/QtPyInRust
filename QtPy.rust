use qt_widgets::{
    QApplication, QMainWindow, QAction, QFileDialog, QMessageBox, QTextEdit, QVBoxLayout, QWidget,
    QTabWidget, QPushButton, QLabel, QSlider, QScrollArea, QInputDialog, QGridLayout, QLineEdit,
    QSplitter, q_application, qt_core::{QString, QIcon, q_color::QColor, q_palette::QPalette, q_widget::QWidget, q_palette::QPaletteColor}
};
use std::fs;
use std::path::Path;
use std::process::Command;

struct QtPy {
    main_window: QMainWindow,
    tab_widget: QTabWidget,
    search_box: QLineEdit,
    return_button: QPushButton,
    next_button: QPushButton,
    splitter: QSplitter,
    scroll_area: QScrollArea,
    previous_directories: Vec<String>,
    next_directories: Vec<String>,
    current_file_path: Option<String>,
    directory_path: Option<String>,
    current_tab: Option<usize>,
}

impl QtPy {
    fn new() -> Self {
        unsafe {
            let app = QApplication::new();
            let mut main_window = QMainWindow::new_0a();

            main_window.set_window_title(&QString::from("QtPy IDE 1.8"));
            main_window.set_geometry_4a(100, 100, 1350, 750);

            let mut tab_widget = QTabWidget::new_0a();
            tab_widget.set_tabs_closable(true);

            let return_button = QPushButton::from_q_string(&QString::from("Return To The Previous Directory"));
            let next_button = QPushButton::from_q_string(&QString::from("Go To The Next Directory"));
            let search_box = QLineEdit::new();
            let search_button = QPushButton::from_q_string(&QString::from("Search"));
            let scroll_area = QScrollArea::new();

            let mut central_widget_layout = QGridLayout::new_0a();
            let mut splitter = QSplitter::new_q_orientation(Qt::Horizontal);
            splitter.add_widget(&tab_widget);
            splitter.add_widget(&scroll_area);

            central_widget_layout.add_widget_5a(&return_button, 0, 0, 1, 1);
            central_widget_layout.add_widget_5a(&next_button, 0, 1, 1, 1);
            central_widget_layout.add_widget_5a(&splitter, 1, 0, 1, 2);
            central_widget_layout.add_widget_5a(&search_box, 2, 0, 1, 1);
            central_widget_layout.add_widget_5a(&search_button, 2, 1, 1, 1);

            let central_widget = QWidget::new_0a();
            central_widget.set_layout(&central_widget_layout);
            main_window.set_central_widget(&central_widget);

            QtPy {
                main_window,
                tab_widget,
                search_box,
                return_button,
                next_button,
                splitter,
                scroll_area,
                previous_directories: Vec::new(),
                next_directories: Vec::new(),
                current_file_path: None,
                directory_path: None,
                current_tab: None,
            }
        }
    }

    fn create_menu(&mut self) {
        unsafe {
            let menu_bar = self.main_window.menu_bar();

            let file_menu = menu_bar.add_menu(&QString::from("File"));

            let open_act = QAction::from_q_string_q_object(&QString::from("Open"), self.main_window.static_cast_mut());
            open_act.triggered().connect(&self.main_window.slot().open_file());
            file_menu.add_action(&open_act);

            let save_act = QAction::from_q_string_q_object(&QString::from("Save"), self.main_window.static_cast_mut());
            save_act.triggered().connect(&self.main_window.slot().save_file());
            file_menu.add_action(&save_act);

            let save_as_act = QAction::from_q_string_q_object(&QString::from("Save As"), self.main_window.static_cast_mut());
            save_as_act.triggered().connect(&self.main_window.slot().save_file_as());
            file_menu.add_action(&save_as_act);

            let exit_act = QAction::from_q_string_q_object(&QString::from("Exit"), self.main_window.static_cast_mut());
            exit_act.triggered().connect(&self.main_window.slot().close());
            file_menu.add_action(&exit_act);

            let run_menu = menu_bar.add_menu(&QString::from("Run"));
            let run_act = QAction::from_q_string_q_object(&QString::from("Run (html)"), self.main_window.static_cast_mut());
            run_act.triggered().connect(&self.main_window.slot().run_file());
            run_menu.add_action(&run_act);
        }
    }

    fn create_toolbar(&mut self) {
        unsafe {
            let toolbar = self.main_window.add_tool_bar(&QString::from("Toolbar"));

            let plus_button = QAction::from_q_string_q_object(&QString::from("Create New Tab"), self.main_window.static_cast_mut());
            plus_button.triggered().connect(&self.main_window.slot().create_new_tab());
            toolbar.add_action(&plus_button);

            let close_all_button = QAction::from_q_string_q_object(&QString::from("Close All Tabs"), self.main_window.static_cast_mut());
            close_all_button.triggered().connect(&self.main_window.slot().close_all_tabs());
            toolbar.add_action(&close_all_button);

            let open_act = QAction::from_q_string_q_object(&QString::from("Open"), self.main_window.static_cast_mut());
            open_act.triggered().connect(&self.main_window.slot().open_file());
            toolbar.add_action(&open_act);

            let save_act = QAction::from_q_string_q_object(&QString::from("Save"), self.main_window.static_cast_mut());
            save_act.triggered().connect(&self.main_window.slot().save_file());
            toolbar.add_action(&save_act);

            let save_as_act = QAction::from_q_string_q_object(&QString::from("Save As"), self.main_window.static_cast_mut());
            save_as_act.triggered().connect(&self.main_window.slot().save_file_as());
            toolbar.add_action(&save_as_act);

            let open_directory_button = QAction::from_q_string_q_object(&QString::from("Open Directory"), self.main_window.static_cast_mut());
            open_directory_button.triggered().connect(&self.main_window.slot().open_directory());
            toolbar.add_action(&open_directory_button);

            let label = QLabel::from_q_string(&QString::from("Text Size:"));
            toolbar.add_widget(&label);

            let slider = QSlider::from_q_orientation_q_widget(Qt::Horizontal, self.main_window.static_cast_mut());
            slider.set_minimum(1);
            slider.set_maximum(40);
            slider.set_value(12);
            toolbar.add_widget(&slider);
        }
    }

    fn create_notebook(&mut self) {
        unsafe {
            self.tab_widget = QTabWidget::new_0a();
            self.tab_widget.set_tabs_closable(true);
        }
    }

    fn create_directory_opener(&mut self) {
        unsafe {
            self.scroll_area = QScrollArea::new();
            self.scroll_area.set_widget_resizable(true);
        }
    }

    fn open_directory(&mut self) {
        unsafe {
            let directory_path = QFileDialog::get_existing_directory_2a(&self.main_window, &QString::from("Open Directory"));
            if !directory_path.is_empty() {
                self.previous_directories.push(directory_path.to_std_string());
                self.return_button.set_visible(true);
                self.refresh_file_list();
            }
        }
    }

    fn refresh_file_list(&mut self) {
        unsafe {
            let layout = self.scroll_area.layout();
            if layout != 0 {
                for i in 0..layout.count() {
                    let item = layout.item_at(i);
                    if item != 0 {
                        item.widget().delete_later();
                    }
                }
            }

            let mut file_list = Vec::new();
            let mut folder_list = Vec::new();
            if let Some(ref path) = self.directory_path {
                for entry in fs::read_dir(path).unwrap() {
                    let entry = entry.unwrap();
                    let entry_path = entry.path();
                    if entry_path.is_file() {
                        file_list.push(entry.file_name().into_string().unwrap());
                    } else if entry_path.is_dir() {
                        folder_list.push(entry.file_name().into_string().unwrap());
                    }
                }
            }

            for folder_name in folder_list {
                let folder_button = QPushButton::from_q_string(&QString::from(format!("{}  (Folder)", folder_name).as_str()));
                folder_button.clicked().connect(&self.main_window.slot().open_folder(folder_name));
                self.scroll_area.layout().add_widget(&folder_button);
            }

            for file_name in file_list {
                let file_button = QPushButton::from_q_string(&QString::from(file_name.as_str()));
                file_button.clicked().connect(&self.main_window.slot().open_selected_file(file_name));
                self.scroll_area.layout().add_widget(&file_button);
            }
        }
    }

    fn open_folder(&mut self, folder_name: String) {
        let folder_path = Path::new(self.directory_path.as_ref().unwrap()).join(folder_name);
        self.previous_directories.push(self.directory_path.take().unwrap());
        self.directory_path = Some(folder_path.to_str().unwrap().to_string());
        self.refresh_file_list();
    }

    fn open_selected_file(&mut self, file_name: String) {
        let file_path = Path::new(self.directory_path.as_ref().unwrap()).join(&file_name);
        let tab_name = QInputDialog::get_text_4a(&self.main_window, &QString::from("Open File"), &QString::from(format!("In which tab do you want to open '{}'", file_name).as_str()), QLineEdit::Normal, &QString::from(""), &mut true);
        if !tab_name.is_empty() {
            let index = self.tab_widget.current_index();
            if index != -1 {
                self.tab_widget.remove_tab(index);
            }
            let mut text_edit = QTextEdit::new_0a();
            text_edit.set_plain_text(&QString::from(fs::read_to_string(file_path).unwrap().as_str()));
            self.tab_widget.add_tab(&text_edit, &tab_name);
            self.current_tab = Some(self.tab_widget.index_of(&text_edit));
            self.current_file_path = Some(file_path.to_str().unwrap().to_string());
        }
    }

    fn open_file(&mut self) {
        let file_path = QFileDialog::get_open_file_name_2a(&self.main_window, &QString::from("Open File"));
        if !file_path.is_empty() {
            let content = fs::read_to_string(file_path.to_std_string()).unwrap();
            let mut text_edit = QTextEdit::new_0a();
            text_edit.set_plain_text(&QString::from(content.as_str()));
            self.tab_widget.add_tab(&text_edit, &QString::from(Path::new(&file_path.to_std_string()).file_name().unwrap().to_str().unwrap()));
            self.current_file_path = Some(file_path.to_std_string());
            self.current_tab = Some(self.tab_widget.index_of(&text_edit));
        }
    }

    fn save_file(&mut self) {
        if let Some(current_tab) = self.current_tab {
            let text_edit: &QTextEdit = self.tab_widget.widget(current_tab).dynamic_cast();
            let content = text_edit.to_plain_text().to_std_string();
            if let Some(ref current_file_path) = self.current_file_path {
                fs::write(current_file_path, content).unwrap();
            } else {
                self.save_file_as();
            }
        } else {
            QMessageBox::warning(&self.main_window, &QString::from("Save Error"), &QString::from("Please select a tab to save."));
        }
    }

    fn save_file_as(&mut self) {
        if let Some(current_tab) = self.current_tab {
            let file_path = QFileDialog::get_save_file_name_2a(&self.main_window, &QString::from("Save File"), &QString::from(""), &QString::from("All files (*)"));
            if !file_path.is_empty() {
                let text_edit: &QTextEdit = self.tab_widget.widget(current_tab).dynamic_cast();
                let content = text_edit.to_plain_text().to_std_string();
                fs::write(file_path.to_std_string(), content).unwrap();
                self.current_file_path = Some(file_path.to_std_string());
            }
        } else {
            QMessageBox::warning(&self.main_window, &QString::from("Save Error"), &QString::from("Please select a tab to save."));
        }
    }

    fn run_file(&mut self) {
        if let Some(ref directory_path) = self.directory_path {
            let selected_file = QInputDialog::get_text_4a(&self.main_window, &QString::from("Run File"), &QString::from("Enter the file name to run:"), QLineEdit::Normal, &QString::from(""), &mut true);
            if !selected_file.is_empty() {
                let file_path = Path::new(directory_path).join(selected_file.to_std_string());
                let file_extension = file_path.extension().unwrap().to_str().unwrap().to_lowercase();
                if file_extension == "html" {
                    Command::new("xdg-open").arg(file_path.to_str().unwrap()).spawn().unwrap();
                } else if file_extension == "py" {
                    QMessageBox::information(&self.main_window, &QString::from("Run Info"), &QString::from("You need to run Python files manually."));
                } else {
                    QMessageBox::warning(&self.main_window, &QString::from("Run Error"), &QString::from(format!("Unsupported file type: {}", file_extension).as_str()));
                }
            }
        } else {
            QMessageBox::warning(&self.main_window, &QString::from("Run Error"), &QString::from("Please open a directory first."));
        }
    }

    fn create_new_tab(&mut self) {
        let tab_name = QInputDialog::get_text_4a(&self.main_window, &QString::from("New Tab"), &QString::from("Enter the name for the new tab:"), QLineEdit::Normal, &QString::from(""), &mut true);
        if !tab_name.is_empty() {
            let mut text_edit = QTextEdit::new_0a();
            self.tab_widget.add_tab(&text_edit, &QString::from(format!("{}*", tab_name.to_std_string()).as_str()));
            self.current_tab = Some(self.tab_widget.index_of(&text_edit));
        }
    }

    fn search(&self) {
        let query = self.search_box.text().to_std_string();
        if !query.is_empty() {
            Command::new("xdg-open").arg(format!("https://search.brave.com/search?q={}", query)).spawn().unwrap();
        } else {
            QMessageBox::warning(&self.main_window, &QString::from("Warning"), &QString::from("Please enter a search query."));
        }
    }

    fn close_all_tabs(&mut self) {
        self.tab_widget.clear();
    }

    fn close_tab(&mut self, index: usize) {
        self.tab_widget.remove_tab(index as i32);
    }

    fn close_directory(&mut self) {
        unsafe {
            self.scroll_area.layout().remove_widget(self.scroll_area.widget());
            self.scroll_area.widget().delete_later();
            self.create_directory_opener();
        }
    }
}

fn main() {
    let qtpy = QtPy::new();
    qtpy.main_window.show();
    q_application::exec();
}

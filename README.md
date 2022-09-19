# ios_filemanager

```swift
//
//  ContentView.swift
//  ios_file_manager
//
//  Created by paige shin on 2022/09/19.
//

import SwiftUI

class LocalFileManager {
    
    static let instance = LocalFileManager()
    
    let folderName = "My_App_Images"
    
    init() {
        self.createFolderIfNeeded()
    }
    
    func createFolderIfNeeded() {
        guard
            let path = FileManager
                .default
                .urls(for: .cachesDirectory, in: .userDomainMask)
                .first?
                .appendingPathComponent("My_App_Images")
                .path(percentEncoded: true) else {
            return
        }
        if !FileManager.default.fileExists(atPath: path) {
            do {
                try FileManager.default.createDirectory(atPath: path, withIntermediateDirectories: true, attributes: nil)
                print("Success creating folder.")
            } catch let error {
                print("Error creating folder. \(error)")
            }
        }
    }
    
    func deleteFolder() {
        guard
            let path = FileManager
                .default
                .urls(for: .cachesDirectory, in: .userDomainMask)
                .first?
                .appendingPathComponent("My_App_Images")
                .path(percentEncoded: true) else {
            return
        }
        do {
            try FileManager.default.removeItem(atPath: path)
            print("Succeed deleting folder")
        } catch {
            print("Error deleting directory")
        }
    }
    
    func differentFolderType() {
        
        /**
         * Get Directory
         * @param for: directory
         * @param in:  position of the files where it will be saved.. user or local machine
         */
        
        /**
         * .documentDirectory: Only documents and other data that is user-generated, or that otherwise be recreated by your application, should be stored in <Application_Home>/Documents directory and will be automatically backed up by iCloud.
         * .cachesDirectory: Data that can be downloaded again or regenerated should be stored in the <Application_Home>/Library/Caches directory. Examples of files you should put in the Caches directory include database cahe files and downloadable content, such as that used by megazine, newspaper, and map applications
         * FileManager.default.temporaryDirectory: Only temporary files
         */
        
        let directory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)
        
        let directory2 = FileManager.default.urls(for: .cachesDirectory, in: .userDomainMask)
        
        let directory3 = FileManager.default.temporaryDirectory
        
        print(directory) // prints out path
        print(directory2)
        print(directory3)
    }
    
    func saveImage(image: UIImage, name: String) {
        
        guard let data = image.jpegData(compressionQuality: 1.0) else {
            print("Error getting data.")
            return
        }
        
        guard let path = FileManager
            .default
            .urls(for: .cachesDirectory, in: .userDomainMask)
            .first?
            .appendingPathComponent(self.folderName)
            .appending(component: "\(name).jpeg") else {
                print("Error getting path.")
                return
            }
        
        do {
            try data.write(to: path)
            print("Success saving!")
        } catch let error {
            print("Error saving. \(error)")
        }
    }
    
    func getImage(name: String) -> UIImage? {
        guard let path = FileManager
            .default
            .urls(for: .cachesDirectory, in: .userDomainMask)
            .first?
            .appendingPathComponent(self.folderName)
            .appending(component: "\(name).jpeg") else {
                print("Error getting path.")
                return nil
            }
        guard FileManager.default.fileExists(atPath: path.path(percentEncoded: true)) else {
            print("No file exists")
            return nil
        }
        return UIImage(contentsOfFile: path.path(percentEncoded: true))
    }
    
    func deleteImage(name: String) {
        guard let path = FileManager
            .default
            .urls(for: .cachesDirectory, in: .userDomainMask)
            .first?
            .appendingPathComponent(self.folderName)
            .appending(component: "\(name).jpeg") else {
                print("Error getting path.")
                return
            }
        do {
            try FileManager.default.removeItem(at: path)
            print("Successfully deleted")
        } catch let error {
            print("Error deleting image> \(error)")
        }
    }
    
}

class FileManagerViewModel: ObservableObject {
    
    @Published var image: UIImage? = nil
    private let imageName: String = "images"
    private let manager = LocalFileManager.instance
    
    init() {
        self.getImageFromAssetsFolder()
//        self.getImageFromFileManager()
    }
    
    func getImageFromAssetsFolder() {
        self.image = UIImage(named: imageName)
    }
    
    func getImageFromFileManager() {
        self.image = self.manager.getImage(name: self.imageName)
    }
    
    func saveImage() {
        guard let image = image else { return }
        self.manager.saveImage(image: image, name: self.imageName)
    }
    
    func deleteImage() {
        self.manager.deleteFolder()
        self.manager.deleteImage(name: self.imageName)
    }
    
}

struct ContentView: View {
    
    @StateObject var vm = FileManagerViewModel()
    
    var body: some View {
        NavigationView {
            VStack {
                if let image = vm.image {
                    Image(uiImage: image)
                        .resizable()
                        .scaledToFill()
                        .frame(width: 200, height: 200)
                        .clipped()
                        .cornerRadius(10)
                }
                
                HStack {
                    Button {
                        self.vm.saveImage()
                    } label: {
                        Text("Save to File Manager")
                            .foregroundColor(.white)
                            .font(.headline)
                            .padding()
                            .padding(.horizontal)
                            .background(Color.blue)
                            .cornerRadius(10)
                    }


                    Button {
                        self.vm.deleteImage()
                    } label: {
                        Text("Delete From File Manager")
                            .foregroundColor(.white)
                            .font(.headline)
                            .padding()
                            .padding(.horizontal)
                            .background(Color.red)
                            .cornerRadius(10)
                    }
                }
          
                
                Spacer()
            }
            .navigationTitle("File Manager")
        }
    }
}

struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}

```

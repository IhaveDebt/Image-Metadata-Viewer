package main

import (
	"flag"
	"fmt"
	"log"
	"os"
	"path/filepath"
	"time"

	"github.com/rwcarlsen/goexif/exif"
)

func printBasicInfo(path string) {
	fi, err := os.Stat(path)
	if err != nil {
		fmt.Println("stat error:", err)
		return
	}
	fmt.Println("File:", filepath.Base(path))
	fmt.Println("Size:", fi.Size(), "bytes")
	fmt.Println("Modified:", fi.ModTime().Format(time.RFC3339))
}

func printExif(path string) {
	f, err := os.Open(path)
	if err != nil {
		fmt.Println("open error:", err)
		return
	}
	defer f.Close()
	x, err := exif.Decode(f)
	if err != nil {
		fmt.Println("no EXIF or decode error:", err)
		return
	}
	x.Walk(exifWalker{})
}

type exifWalker struct{}

func (exifWalker) Walk(name exif.FieldName, tag *exif.Tag) error {
	fmt.Printf("%-25s : %v\n", name, tag)
	return nil
}

func main() {
	file := flag.String("file", "", "image file path (JPEG with EXIF recommended)")
	flag.Parse()

	if *file == "" {
		fmt.Println("Usage: -file=path/to/photo.jpg")
		return
	}
	printBasicInfo(*file)
	fmt.Println("\nEXIF data:")
	printExif(*file)
}

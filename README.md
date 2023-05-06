# ocarina
# the program reads piano notes and adapts them for ocarina

from PIL import Image, ImageDraw
import pygame
from music21 import midi

# определяем размеры изображения окарины
image_size = (300, 600)

# загружаем картинку окарины
okarina_image = Image.open('okarina.png')

# создаем новое изображение
result_image = Image.new('RGBA', image_size, (255, 255, 255, 255))

# определяем координаты отверстий
hole_coords = [
    [(75, 100), (225, 250)],  # первое отверстие
    [(25, 250), (275, 400)],  # второе отверстие
    [(75, 400), (225, 550)],  # третье отверстие
    [(0, 0), (0, 0)],  # четвертое отверстие (закрыто)
    [(0, 0), (0, 0)],  # пятое отверстие (закрыто)
    [(0, 0), (0, 0)]   # шестое отверстие (закрыто)
]

# загружаем MIDI-файл с нотами фортепиано
midi_file = midi.MidiFile()
midi_file.open('piano_notes.mid')
midi_file.read()
midi_file.close()

# создаем Pygame-окно для воспроизведения звуков нот
pygame.mixer.init()

# обходим все ноты в MIDI-файле
for event in midi_file.events:
    if isinstance(event, midi.NoteOnEvent):
        # получаем номер ноты
        note_number = event.pitch

        # адаптируем номер ноты для окарины
        hole_number = note_number % 6

        # закрашиваем соответствующее отверстие на изображении окарины
        hole_coord = hole_coords[hole_number]
        draw = ImageDraw.Draw(result_image)
        draw.rectangle(hole_coord, fill=(0, 0, 0, 255))

        # воспроизводим звук ноты
        pygame.mixer.music.load(f'piano_notes/{note_number}.mp3')
        pygame.mixer.music.play()

        # ждем, пока звук закончится
        while pygame.mixer.music.get_busy():
            pass

# сохраняем результат
result_image = Image.alpha_composite(okarina_image, result_image)
result_image.save('okarina_notes.png')

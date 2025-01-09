#!/Users/nathanbraun/code/github.com/nathanbraun/apy-zk-note-sync/venv/bin/python

import argparse
import glob
import os
import io
from ruamel.yaml import YAML, YAMLError
from pathlib import Path
from apyanki.anki import Anki
from apyanki.config import cfg
from apyanki.console import console
from apyanki.note import Note

parser = argparse.ArgumentParser(
  description="Add unsynced zk notes to Anki.")

parser.add_argument(
    '-d', '--note-directory',
    help="Note directory. Defaults to ~/notes.",
    required=False)

args = vars(parser.parse_args())

note_directory = args['note_directory']

if note_directory is None:
    note_directory = os.path.expanduser('~/notes')

# Define the pattern for the files we're interested in
pattern = os.path.join(note_directory, '*-anki.wiki')

# Use ruamel.yaml to handle YAML
yaml = YAML()

def is_synced(file_path):
    try:
        with open(file_path, 'r', encoding='utf-8') as file:
            if file.readline().strip() != '---':
                return False
            
            header_lines = []
            for line in file:
                if line.strip() == '---':
                    break
                header_lines.append(line)
            
            try:
                yaml_content = yaml.load(''.join(header_lines))
            except YAMLError as e:
                console.print(f"YAML parsing error in {file_path}: {e}")
                return False

            return yaml_content.get('synced', False) is True
    except IOError as e:
        console.print(f"Error reading {file_path}: {e}")
        return False

def set_synced(file_path):
    try:
        with open(file_path, 'r+', encoding='utf-8') as file:
            lines = file.readlines()
            
            if lines[0].strip() == '---':
                yaml_lines = []
                for i, line in enumerate(lines[1:], start=1):
                    if line.strip() == '---':
                        try:
                            yaml_content = yaml.load(''.join(yaml_lines))
                        except YAMLError as e:
                            console.print(f"YAML parsing error in {file_path}: {e}")
                            return

                        break
                    yaml_lines.append(line)

                # Update the 'synced' field
                yaml_content['synced'] = True

                # Use a StringIO stream to capture the YAML dump
                string_stream = io.StringIO()
                yaml.dump(yaml_content, string_stream)
                new_yaml_lines = string_stream.getvalue().splitlines(True)

                # Update the file's content
                updated_content = lines[:1] + new_yaml_lines + lines[i:]

                # Move the file cursor to the beginning and truncate the file
                file.seek(0)
                file.truncate()

                # Write the updated content back
                file.writelines(updated_content)
    except IOError as e:
        console.print(f"Error writing to {file_path}: {e}")

def add_notes_from_file(file_path: Path, tags: str = "", deck: str = "") -> bool:
    """
    Add notes to Anki from a specified markdown file.

    :param file_path: Path to markdown file containing notes.
    :param tags: Tags to be added to each note (optional).
    :param deck: Deck to add notes to (optional).
    :return: True if notes were successfully added, otherwise False.
    """
    try:
        with Anki(**cfg) as a:
            notes = a.add_notes_from_file(str(file_path), tags, deck)
            _added_notes_postprocessing(a, notes)
            a.sync()
            return len(notes) > 0
    except Exception as e:
        console.print(f"Error adding notes from {file_path}: {e}")
        return False

def _added_notes_postprocessing(a: Anki, notes: list[Note]) -> None:
    """Handle common postprocessing after adding notes."""
    n_notes = len(notes)
    if n_notes == 0:
        console.print("No notes added")
        return

    decks = [a.col.decks.name(c.did) for n in notes for c in n.n.cards()]
    n_decks = len(decks)
    if n_decks == 0:
        console.print("No notes added")
        return

    if a.n_decks > 1:
        if n_notes == 1:
            console.print(f"Added note to deck: {decks[0]}")
        elif n_decks > 1:
            console.print(f"Added {n_notes} notes to {n_decks} different decks")
        else:
            console.print(f"Added {n_notes} notes to deck: {decks[0]}")
    else:
        console.print(f"Added {n_notes} notes")

    for note in notes:
        cards = note.n.cards()
        console.print(f"* nid: {note.n.id} (with {len(cards)} cards)")
        for card in note.n.cards():
            console.print(f"  * cid: {card.id}")

# Get a list of matching files
matching_files = glob.glob(pattern)

# Filter the list of matching files to exclude those that are synced
unsynced_files = [f for f in matching_files if not is_synced(f)]

# Process each unsynced file
for file_path in unsynced_files:
    success = add_notes_from_file(Path(file_path))
    if success:
        set_synced(file_path)
        print(f"Updated and synced {file_path}")
    else:
        print(f"Failed to add notes from {file_path}")

